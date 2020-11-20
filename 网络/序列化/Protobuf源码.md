

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

