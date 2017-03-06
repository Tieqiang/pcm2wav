[参考的原文链接](http://blog.csdn.net/h3c4lenovo/article/details/8113374)

## 关于riff/wav/pcm/raw/mp3

### riff
一种文件描述格式

### wav
wav文件就采用了riff描述，其前44字节就是riff描述内容

### pcm
- 媒体数据的元数据，直接记录声音内容
- wav = riff + pcm
- raw = pcm

### mp3
一种压缩编码，将pcm通过算法压缩

## 采样率/比特率/通道数/采样值

### 采样率
模拟信号采集频率，即每秒样本数，采样率越高声音还原越真实

### 比特率
模拟信号转为数字信号后的采样率，常见8位与16位

### 通道
常见有单通道和双通道，单通道每样本占用8/16位，双通道每样本占用16/32位，
且高8/16位是左声道，低8/16位是右声道

## 如何实现音频合成
1. 录制的声音是pcm即元数据，我们需要与另外一个声音文件合并，这样就必须将另外一个文件解码为pcm数据
2. 获得录音与伴奏的pcm数据之后进行混合，简单的来说就是算每位字节的平均值
3. 得到混合后的音频pcm之后进行再编码，比如以最终生成wav格式为例，只需要在pcm数据前加入44字节的riff描述即可

## 可能遇到的问题
1. 录音失真
在测试的过程中发现录音文件播放速度非常快
经分析发现是wav文件头描述与文件实际内容不匹配导致
比如采用44100采样率，单声道，16位采样值录音，但是riff中写的是双声道，算出来的播放时间比实际时间少了一半，所以播放速度加倍
得出来的结论就是riff描述内容一定要与pcm实际内容匹配，否则就会导致播放声音异常
由此也可以推断只要文件能播，但是播放效果异常就说明多半是riff与pcm类型不匹配导致的

2. 部分文件不支持
出现概率很小，可能是文件本身编码问题

3. 录音过程耗时
AudioRecord.read的时候会阻塞耗时，所以会出现MediaPlayer播放完了，录音只录到一半

4. 混合算法
理论上混合算法就是每位字节相加除以位数求平均数即可，但前提条件是两个文件的通道数&采样值必须一样
之前也解释过通道数与采样值的作用，所以很好推断如果采样值8bit要合16bit就先要byte转short再叠加求均数
如果是单通道要合双通道就写算法加在前半位或后半位
双通道转单通道就前半位与后半位叠加后算平均数之后再与单通道叠加算平均数

5. Android设备只能以单声道的形式录制，即使传入双声道的参数运行不报错，也只能将生成的PCM当作单声道文件处理