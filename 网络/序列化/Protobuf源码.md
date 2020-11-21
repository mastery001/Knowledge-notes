[protobuf 编码实现解析（java）](https://www.cnblogs.com/onlysun/p/4574850.html)

[issues-int32 promoted 64-bit](https://github.com/apple/swift-protobuf/issues/37)

[protobuf encoding](https://developers.google.com/protocol-buffers/docs/encoding)

[使用64位-补码的猜测](https://stackoverflow.com/questions/765916/is-there-ever-a-good-time-to-use-int32-instead-of-sint32-in-google-protocol-buff)

```java
public final void writeInt32NoTag(int value) throws IOException {
  if (value >= 0) {
    writeUInt32NoTag(value);
  } else {
    // 猜测可能是补码导致的，protobuf是采用2的补码方式来表示负整数，感觉是没有使用符号位缩减的特性(即long强转为int)，估计是最初开发的时候忽略了这个问题，后面不好修复；在2.5版本中，读取int都是直接丢弃大于32个字节的数字
    // Must sign-extend.
    writeUInt64NoTag(value);
  }
}
```



【2.5.0的代码】

```java
/**
 * 虽然写入也是10个字节，但是读取的时候就直接丢弃了后面的字节
 * Read a raw Varint from the stream.  If larger than 32 bits, discard the
 * upper bits.
 */
public int readRawVarint32() throws IOException {
  byte tmp = readRawByte();
  if (tmp >= 0) {
    return tmp;
  }
  int result = tmp & 0x7f;
  if ((tmp = readRawByte()) >= 0) {
    result |= tmp << 7;
  } else {
    result |= (tmp & 0x7f) << 7;
    if ((tmp = readRawByte()) >= 0) {
      result |= tmp << 14;
    } else {
      result |= (tmp & 0x7f) << 14;
      if ((tmp = readRawByte()) >= 0) {
        result |= tmp << 21;
      } else {
        result |= (tmp & 0x7f) << 21;
        result |= (tmp = readRawByte()) << 28;
        if (tmp < 0) {
          // Discard upper 32 bits.
          for (int i = 0; i < 5; i++) {
            if (readRawByte() >= 0) {
              return result;
            }
          }
          throw InvalidProtocolBufferException.malformedVarint();
        }
      }
    }
  }
  return result;
}
```





```java
public int readRawVarint32() throws IOException {
  // See implementation notes for readRawVarint64
  fastpath:
  {
    int tempPos = pos;

    if (limit == tempPos) {
      break fastpath;
    }

    final byte[] buffer = this.buffer;
    int x;
    // 如果1个字节就能表示结束读取
    if ((x = buffer[tempPos++]) >= 0) {
      pos = tempPos;
      return x;
      // 不满足8个字节，则直接走兜底读取的逻辑
    } else if (limit - tempPos < 9) {
      break fastpath;
      // 以下几行为判断2个、3个、4个字节是否能表示这个数
      // 小于0代表不需要再往下读取了，x的符号位为1，buffer[tempPos++]的符号位为0，异或后为0
    } else if ((x ^= (buffer[tempPos++] << 7)) < 0) {
      x ^= (~0 << 7);
    } else if ((x ^= (buffer[tempPos++] << 14)) >= 0) {
      x ^= (~0 << 7) ^ (~0 << 14);
    } else if ((x ^= (buffer[tempPos++] << 21)) < 0) {
      x ^= (~0 << 7) ^ (~0 << 14) ^ (~0 << 21);
    } else {
      // 28位表示不够，继续向下读1个字节
      int y = buffer[tempPos++];
      x ^= y << 28;
      x ^= (~0 << 7) ^ (~0 << 14) ^ (~0 << 21) ^ (~0 << 28);
      // 如果后面5个字节都小于0，则代表是64位long类型的数字
      // 如果单纯写int或long类型的数字不可能会满足这个条件，所以当满足时会抛出异常
      if (y < 0
          && buffer[tempPos++] < 0
          && buffer[tempPos++] < 0
          && buffer[tempPos++] < 0
          && buffer[tempPos++] < 0
          && buffer[tempPos++] < 0) {
        break fastpath; // Will throw malformedVarint()
      }
    }
    pos = tempPos;
    return x;
  }
  return (int) readRawVarint64SlowPath();
}
```

