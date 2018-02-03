---
title: BMP压缩成JPEG过程实现与分析
date: 2017-06-13
tags:
    - 图像处理
    - C/C++
    - Algo
    - Math
---

[GitHub](https://github.com/mk43/Algo-Math/tree/master/bmp2jpeg)

---
### 前言
由于最近做了图像相关的学习，所以想再深入点，但是自己的知识有限，目前只能把自己所学的通过这个小小的实验来加深理解。参考大牛的 Blog 加上自己亲手实践，写了这篇文章。以后还会继续添加图像处理的相关知识。
<img src="https://raw.githubusercontent.com/mk43/Algo-Math/master/bmp2jpeg/y.jpeg" width="240" height="240"/> <img src="https://raw.githubusercontent.com/mk43/Algo-Math/master/bmp2jpeg/b.jpeg" width="240" height="240"/> <img src="https://raw.githubusercontent.com/mk43/Algo-Math/master/bmp2jpeg/r.jpeg" width="240" height="240"/>

<!-- more -->

### [BMP介绍](https://www.cnblogs.com/Matrix_Yao/archive/2009/12/02/1615295.html)

- BMP文件头
文件头主要是包含一个文件的ID信息，所以BMP的文件头自然也是说明自己的文件格式，文件大小等信息，一般是14位表示。意义如下图所示：
![](bmp_1.png)

- 位图信息头
信息头主要是对图片特征的描述，比如说宽高，像素，压缩方式等，一般是40位。主要介绍如下表：
![](bmp_2.png)

- 调色板
调色板是可选的，使用索引来表示图像，调色板就是索引与其对应的颜色的映射表，这次实验选用的是24bit的图片。
- 位图数据
这里就是存储图片的内容了。

### [JPEG介绍](https://www.cnblogs.com/lakeone/p/3596996.html)

JPEG是有损压缩编码下的一种图片格式，目前压缩效果好，应用广泛。其原理主要是将传统的RGB模式下的图片转化成YCbCr格式。因为人眼的结构问题对亮度更加敏感，所以可以将亮度和色度分离开来，对色度可进行较大的舍弃从而进行较大程度的压缩而对视觉不造成太大影响。既然是压缩而成的格式，那必然有解压缩过程，而解压缩所以依赖的量化表和哈夫曼表自然要记录下来。所以和BMP对比自然而然头结构出来了，而且要比BMP复杂。下面只给出部分信息：
![](jpeg_1.png)

### BMP 读取
- 构建BMP的文件头和头信息结构体

```c
//BMP 文件格式【文件头和头部信息】
typedef struct {
		unsigned short	bfType;
		unsigned int	bfSize;
		unsigned short	bfReserved1;
		unsigned short	bfReserved2;
		unsigned int	bfOffBits;
} BITMAPFILEHEADER;

typedef struct {
		unsigned int	biSize;
		int				biWidth;
		int				biHeight;
		unsigned short	biPlanes;
		unsigned short	biBitCount;
		unsigned int	biCompression;
		unsigned int	biSizeImage;
		int				biXPelsPerMeter;
		int				biYPelsPerMeter;
		unsigned int	biClrUsed;
		unsigned int	biClrImportant;
} BITMAPINFOHEADER;
```

- 图片校验

```c
//打开文件
FILE* fp = fopen(fileName, "rb");
if(fp==0) {
	return false;
}

BITMAPFILEHEADER fileHeader;
BITMAPINFOHEADER infoHeader;

// 读取头部 14字节
if(1 != fread(&fileHeader, sizeof(fileHeader), 1, fp)) {
	return false;
}
// 判断是不是BM类型
if(fileHeader.bfType!=0x4D42) {
	return false;
}

// 读取头部信息 40字节
if(1 != fread(&infoHeader, sizeof(infoHeader), 1, fp)) {
	return false;
}
// 判断是不是24位类型。也就是RGB的存储格式
if(infoHeader.biBitCount != 24 || infoHeader.biCompression != 0) {
	return false;
}
int width = infoHeader.biWidth;
int height = infoHeader.biHeight < 0 ? (-infoHeader.biHeight) : infoHeader.biHeight;
// 判断二进制的最后三位是不是000，也就是判断是不是8的倍数
if((width&7) != 0 || (height&7) != 0) {
	return false;
}
```

- 图片内容读取

```c
// RGB三个分量
int bmpSize = width*height*3;

unsigned char* buffer = new unsigned char[bmpSize];
if(buffer == 0) {
	return false;
}

// 将文件指针移到数据区域
fseek(fp, fileHeader.bfOffBits, SEEK_SET);

if(infoHeader.biHeight > 0) {
	for(int i = 0; i < height; i++) {
		// 读取第i行,每此读 3（size） * width （count）大小
		if(width != fread(buffer + (height - 1 - i) * width * 3, 3, width, fp)) {
			delete[] buffer;
			buffer = 0;
			return false;
		}
	}
} else {
	if(width*height != fread(buffer, 3, width*height, fp)) {
		delete[] buffer;
		buffer = 0;
		return false;
	}
}
```

- 存储信息

```c
// 获取宽高和大小
m_rgbBuffer = buffer;
m_width = width;
m_height = height;

fclose(fp);
fp=0;
```

### JPEG 写入
在前期JPEG写入是，要进行一系列准备工作，根据JPEG官方提供的标准量化表和哈夫曼表进行自己的操作得到自己满意的压缩编码。

- 数值表：
![](tdata.png)

- 直流分量表：
![](tdc.png)

- 交流分量表：
![](tac.png)

下面给出具体代码：

- 亮度量化表

```c
// 亮度量化表
const unsigned char Luminance_Quantization_Table[64] = {
	16,  11,  10,  16,  24,  40,  51,  61,
	12,  12,  14,  19,  26,  58,  60,  55,
	14,  13,  16,  24,  40,  57,  69,  56,
	14,  17,  22,  29,  51,  87,  80,  62,
	18,  22,  37,  56,  68, 109, 103,  77,
	24,  35,  55,  64,  81, 104, 113,  92,
	49,  64,  78,  87, 103, 121, 120, 101,
	72,  92,  95,  98, 112, 100, 103,  99
};
```

- 色度量化表

```c
// 色度量化表
const unsigned char Chrominance_Quantization_Table[64] = {
	17,  18,  24,  47,  99,  99,  99,  99,
	18,  21,  26,  66,  99,  99,  99,  99,
	24,  26,  56,  99,  99,  99,  99,  99,
	47,  66,  99,  99,  99,  99,  99,  99,
	99,  99,  99,  99,  99,  99,  99,  99,
	99,  99,  99,  99,  99,  99,  99,  99,
	99,  99,  99,  99,  99,  99,  99,  99,
	99,  99,  99,  99,  99,  99,  99,  99
};
```

- 标准直流分量色度亮度哈夫曼表

```c
const char Standard_DC_Luminance_NRCodes[] = { 0, 0, 7, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0 };
const unsigned char Standard_DC_Luminance_Values[] = { 4, 5, 3, 2, 6, 1, 0, 7, 8, 9, 10, 11 };

const char Standard_DC_Chrominance_NRCodes[] = { 0, 3, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0 };
const unsigned char Standard_DC_Chrominance_Values[] = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11 };
```

- 标准交流分量色度亮度哈夫曼表


```c
const char Standard_AC_Luminance_NRCodes[] = { 0, 2, 1,
	3, 3, 2, 4, 3, 5, 5, 4, 4, 0, 0, 1, 0x7d };
const unsigned char Standard_AC_Luminance_Values[] = {
	0x01, 0x02, 0x03, 0x00, 0x04, 0x11, 0x05, 0x12,
	0x21, 0x31, 0x41, 0x06, 0x13, 0x51, 0x61, 0x07,
	0x22, 0x71, 0x14, 0x32, 0x81, 0x91, 0xa1, 0x08,
	0x23, 0x42, 0xb1, 0xc1, 0x15, 0x52, 0xd1, 0xf0,
	0x24, 0x33, 0x62, 0x72, 0x82, 0x09, 0x0a, 0x16,
	0x17, 0x18, 0x19, 0x1a, 0x25, 0x26, 0x27, 0x28,
	0x29, 0x2a, 0x34, 0x35, 0x36, 0x37, 0x38, 0x39,
	0x3a, 0x43, 0x44, 0x45, 0x46, 0x47, 0x48, 0x49,
	0x4a, 0x53, 0x54, 0x55, 0x56, 0x57, 0x58, 0x59,
	0x5a, 0x63, 0x64, 0x65, 0x66, 0x67, 0x68, 0x69,
	0x6a, 0x73, 0x74, 0x75, 0x76, 0x77, 0x78, 0x79,
	0x7a, 0x83, 0x84, 0x85, 0x86, 0x87, 0x88, 0x89,
	0x8a, 0x92, 0x93, 0x94, 0x95, 0x96, 0x97, 0x98,
	0x99, 0x9a, 0xa2, 0xa3, 0xa4, 0xa5, 0xa6, 0xa7,
	0xa8, 0xa9, 0xaa, 0xb2, 0xb3, 0xb4, 0xb5, 0xb6,
	0xb7, 0xb8, 0xb9, 0xba, 0xc2, 0xc3, 0xc4, 0xc5,
	0xc6, 0xc7, 0xc8, 0xc9, 0xca, 0xd2, 0xd3, 0xd4,
	0xd5, 0xd6, 0xd7, 0xd8, 0xd9, 0xda, 0xe1, 0xe2,
	0xe3, 0xe4, 0xe5, 0xe6, 0xe7, 0xe8, 0xe9, 0xea,
	0xf1, 0xf2, 0xf3, 0xf4, 0xf5, 0xf6, 0xf7, 0xf8,
	0xf9, 0xfa
};

const char Standard_AC_Chrominance_NRCodes[] = { 0, 2, 1,
	2, 4, 4, 3, 4, 7, 5, 4, 4, 0, 1, 2, 0x77 };
const unsigned char Standard_AC_Chrominance_Values[] = {
	0x00, 0x01, 0x02, 0x03, 0x11, 0x04, 0x05, 0x21,
	0x31, 0x06, 0x12, 0x41, 0x51, 0x07, 0x61, 0x71,
	0x13, 0x22, 0x32, 0x81, 0x08, 0x14, 0x42, 0x91,
	0xa1, 0xb1, 0xc1, 0x09, 0x23, 0x33, 0x52, 0xf0,
	0x15, 0x62, 0x72, 0xd1, 0x0a, 0x16, 0x24, 0x34,
	0xe1, 0x25, 0xf1, 0x17, 0x18, 0x19, 0x1a, 0x26,
	0x27, 0x28, 0x29, 0x2a, 0x35, 0x36, 0x37, 0x38,
	0x39, 0x3a, 0x43, 0x44, 0x45, 0x46, 0x47, 0x48,
	0x49, 0x4a, 0x53, 0x54, 0x55, 0x56, 0x57, 0x58,
	0x59, 0x5a, 0x63, 0x64, 0x65, 0x66, 0x67, 0x68,
	0x69, 0x6a, 0x73, 0x74, 0x75, 0x76, 0x77, 0x78,
	0x79, 0x7a, 0x82, 0x83, 0x84, 0x85, 0x86, 0x87,
	0x88, 0x89, 0x8a, 0x92, 0x93, 0x94, 0x95, 0x96,
	0x97, 0x98, 0x99, 0x9a, 0xa2, 0xa3, 0xa4, 0xa5,
	0xa6, 0xa7, 0xa8, 0xa9, 0xaa, 0xb2, 0xb3, 0xb4,
	0xb5, 0xb6, 0xb7, 0xb8, 0xb9, 0xba, 0xc2, 0xc3,
	0xc4, 0xc5, 0xc6, 0xc7, 0xc8, 0xc9, 0xca, 0xd2,
	0xd3, 0xd4, 0xd5, 0xd6, 0xd7, 0xd8, 0xd9, 0xda,
	0xe2, 0xe3, 0xe4, 0xe5, 0xe6, 0xe7, 0xe8, 0xe9,
	0xea, 0xf2, 0xf3, 0xf4, 0xf5, 0xf6, 0xf7, 0xf8,
	0xf9, 0xfa
};
```

- 计算哈夫曼编码

```c
void JpegEncoder::_computeHuffmanTable(const char* nr_codes,
	const unsigned char* std_table, BitString* huffman_table) {
	unsigned char pos_in_table = 0;
	unsigned short code_value = 0;

	for(int k = 1; k <= 16; k++) {
		for(int j = 1; j <= nr_codes[k-1]; j++) {
			huffman_table[std_table[pos_in_table]].value = code_value;
			huffman_table[std_table[pos_in_table]].length = k;
			pos_in_table++;
			code_value++;
		}
		code_value <<= 1;
	}
}
```

- 初始化量化表
根据传入的参数调整量化程度，因为这个量化过程是有损的。所以其结果对图像质量有较大影响。

```c
// 初始化量化表
void JpegEncoder::_initQualityTables(int quality_scale) {
	if(quality_scale <= 0) {
		quality_scale = 1;
	}
	if(quality_scale >= 100) {
		quality_scale = 99;
	}

	for(int i = 0; i < 64; i++) {
		int temp = ((int)(Luminance_Quantization_Table[i] * quality_scale + 50) / 100);
		if (temp <= 0) {
			temp = 1;
		}
		if (temp > 0xFF) {
			temp = 0xFF;
		}

		m_YTable[ZigZag[i]] = (unsigned char)temp;

		temp = ((int)(Chrominance_Quantization_Table[i] * quality_scale + 50) / 100);
		if (temp<=0) {
			temp = 1;
		}
		if (temp>0xFF) {
			temp = 0xFF;
		}
		m_CbCrTable[ZigZag[i]] = (unsigned char)temp;
	}
}
```

- 写文件头
到这里文件头基本已经确定，可以写入JPEG文件了。

- RGB 转化成 YCbCr
每读取一个 8*8 的方块区域，就进行颜色空间转化。转换式和代码如下：

```c
Y= 0.299*R + 0.587*G + 0.114*B
C_b= -0.168*R – 0.331*G + 0.449*B
C_r= 0.5*R – 0.419*G – 0.018*B 


void JpegEncoder::_convertColorSpace(int xPos, int yPos, char* yData, char* cbData, char* crData) {
	for (int y = 0; y < 8; y++) {
		// 跳行
		unsigned char* p = m_rgbBuffer + (y + yPos) * m_width * 3 + xPos * 3;
		for (int x = 0; x < 8; x++) {
			unsigned char B = *p++;
			unsigned char G = *p++;
			unsigned char R = *p++;

			yData[y * 8 + x] = (char)(0.299f * R + 0.587f * G + 0.114f * B - 128);
			// yData[y * 8 + x] = 0;
			cbData[y * 8 + x] = (char)(-0.1687f * R - 0.3313f * G + 0.5f * B );
			// cbData[y * 8 + x] = 0;
			crData[y * 8 + x] = (char)(0.5f * R - 0.4187f * G - 0.0813f * B);
			// crData[y * 8 + x] = 0;
		}
	}
}
```

- DCT变换和量化
DCT变换式和量化代码：
![](dct.png)

```c
// DCT变化 + 量化（未优化）
void JpegEncoder::_forward_DCT(const char* channel_data, short* fdc_data) {
	const float PI = 3.1415926f;
	for(int v = 0; v < 8; v++) {
		for(int u = 0; u < 8; u++) {
			float alpha_u = (u==0) ? 1 / sqrt(8.0f) : 0.5f;
			float alpha_v = (v==0) ? 1 / sqrt(8.0f) : 0.5f;

			float temp = 0.f;
			for(int x = 0; x < 8; x++) {
				for(int y = 0; y < 8; y++) {
					float data = channel_data[y * 8 + x];

					data *= cos((2 * x + 1) * u * PI / 16.0f);
					data *= cos((2 * y + 1) * v * PI / 16.0f);

					temp += data;
				}
			}

			temp *= alpha_u * alpha_v / m_YTable[ZigZag[v * 8 + u]];
			fdc_data[ZigZag[v*8+u]] = (short) ((short)(temp + 16384.5) - 16384);
		}
	}
}
```

- 哈夫曼编码
  + 直流分量差分编码

  ```c
  	// encode DC
	int dcDiff = (int)(DU[0] - prevDC);
	prevDC = DU[0];

	if (dcDiff == 0) {
		outputBitString[index++] = HTDC[0];
	} else {
		BitString bs = _getBitCode(dcDiff);

		outputBitString[index++] = HTDC[bs.length];
		outputBitString[index++] = bs;
	}
  ```

  + 交流分量游长编码

  ```c
  	// encode ACs
	int endPos=63; //end0pos = first element in reverse order != 0
	while((endPos > 0) && (DU[endPos] == 0)) {
		endPos--;
	}

	for(int i = 1; i <= endPos; ) {
		int startPos = i;

		while((DU[i] == 0) && (i <= endPos)) {
			i++;
		}

		int zeroCounts = i - startPos;
		if (zeroCounts >= 16) {
			for (int j = 1; j <= zeroCounts / 16; j++) {
				outputBitString[index++] = SIXTEEN_ZEROS;
			}
			zeroCounts = zeroCounts % 16;
		}

		BitString bs = _getBitCode(DU[i]);

		outputBitString[index++] = HTAC[(zeroCounts << 4) | bs.length];
		outputBitString[index++] = bs;
		i++;
	}
  ```

对三个通道进行以上同样的操作。（DCT变化-哈夫曼编码-写入）

```c
BitString outputBitString[128];
int bitStringCounts;

// Y通道压缩
_forward_DCT(yData, yQuant);
_doHuffmanEncoding(yQuant, prev_DC_Y, m_Y_DC_Huffman_Table, m_Y_AC_Huffman_Table,
	outputBitString, bitStringCounts);
_write_bitstring_(outputBitString, bitStringCounts, newByte, newBytePos, fp);

// Cb通道压缩
_forward_DCT(cbData, cbQuant);
_doHuffmanEncoding(cbQuant, prev_DC_Cb, m_CbCr_DC_Huffman_Table, m_CbCr_AC_Huffman_Table,
	outputBitString, bitStringCounts);
_write_bitstring_(outputBitString, bitStringCounts, newByte, newBytePos, fp);

// Cr通道压缩
_forward_DCT(crData, crQuant);
_doHuffmanEncoding(crQuant, prev_DC_Cr, m_CbCr_DC_Huffman_Table, m_CbCr_AC_Huffman_Table,
	outputBitString, bitStringCounts);
_write_bitstring_(outputBitString, bitStringCounts, newByte, newBytePos, fp);
```

整个流程就是如下图所示：
![](t.png)

### 实验结果

测试图片 pic1.bmp
![](pic1.bmp)

16进制
![](bmp0x.png)

测试代码

```c
const char* inputFileName = "pic1.bmp";

JpegEncoder encoder;
// 读取BMP格式的文件
if(!encoder.readFromBMP(inputFileName)) {
	return 1;
}

// 将BMP格式的文件按照JPEG标准压缩成JPEG文件
if(!encoder.encodeToJPG("out.jpeg", 50)) {
    printf("jpg\n");
	return 1;
}
```

读取的BMP文件信息，大小和尺寸都符合原图
![](bmpinfo.png)

测试结果 out.jpeg
![](ybr.jpeg)

十六进制，可以和标准格式比较确实是通过BMP转成了JPEG格式
![](jpeg0x.png)

![](size.png)
可以看到，压缩效果还是比较比较明显的，但是编码性能不是最好的，没有对数据前期进行优化，效率只是中规中矩。
下面介绍对流程和结果的测试分析
过程流程：
![](flowchart.png)

分析为什么转换成YCbCr域对色域的压缩会让人接受：
从RGB到YCbCr的转换公式我们可以分析出Y所占比重较高，说明应该存储的细节相对较多，和人眼对亮度更加敏感符合。那么事实是否如此？ <br/>
<img src="https://raw.githubusercontent.com/mk43/Algo-Math/master/bmp2jpeg/y.jpeg" width="240" height="240"/> <img src="https://raw.githubusercontent.com/mk43/Algo-Math/master/bmp2jpeg/b.jpeg" width="240" height="240"/> <img src="https://raw.githubusercontent.com/mk43/Algo-Math/master/bmp2jpeg/r.jpeg" width="240" height="240"/>

从左到右依次是 Y（72.8k），Cb（37.4k），Cr（33.9k）分量，从光感上说，明显是Y的灰度图像给出了细节，其它两个分量只是给出色彩，没有细节。接着从大小分析也和我们的预测符合，大概比例是 2：1：1，说明存储的细节越多所需的空间自然越大。
接下来对BMP原始通道RGB加扰动和YCbCr加相同的扰动，对图像的影响又会怎样？

![](rgb.jpeg) ![](ybr.jpeg)
从左到右一次是在RGB通道和YCbCr通道加干扰。可以看到RGB收干扰的程度更大，原因不大好用数学分析，我觉得很可能是RGB通道对干扰是没有减弱直接进入通道转换，而YCrCb则是在色度通道进行压缩了，同时也是对干扰的舍弃，所以效果比较好。

下面分析为什么量化矩阵对结果会有很大影响，可以做一个实验，改变生成量化矩阵的算法，看看结果如何。
![](q1.jpeg) ![](q2.jpeg)

这两张的量化程度不同，但是可以看到的是他们都有或多或少的呈色块显示迹象，所以应该存储的空间应该是很小的。
![](qsize.png)
结果也确实如此，回到问题，我们的量化矩阵没有优化，造成数值过大，在量化过程中，导致过多数为0，也就是那些高频分量，而高频正是细节的体现，失去高频自然就失去了细节。所以量化矩阵的取值直接关系到了生成图像的品质。

以上是我对BMP转换成JPEG的过程分析，同时也辅以代码加以实现和测试。对于JPEG的解码过程那就是过程的逆过程了，但是由于编码是有损的，而且编码表量化表都是有转型损失的，所以解码之后的图像也会有部分损失。着呢个过程和读取解码BMP一样。先读取文件头，接下来初始化表，再就是直接读取数据根据表解码出YCbCr的值，反量化之后通过DCT逆变换还原。

其中的源码是[thecodeway](https://thecodeway.com/blog/?p=522)提供的，欢迎大家去他的 Blog 看看他的图像分析文章，我只是对他的代码加以自己的理解。

最后：如有不足，欢迎指正，共同进步。

多谢阅读

- 参考资料
[1] 足迹 : https://www.cnblogs.com/Matrix_Yao/archive/2009/12/02/1615295.html <br/>
[2] lakeone : https://www.cnblogs.com/lakeone/p/3596996.html <br/>
[3] thecodeway : https://thecodeway.com/blog/?p=522 <br/>
[4] SoC Design Lab http://twins.ee.nctu.edu.tw/courses/soclab_04/lab_hw_pdf/proj1_jpeg_introduction.pdf <br/>