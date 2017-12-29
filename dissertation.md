# 论文题目：中文统计参数语音合成系统的实现与改进
本文正在持续更新中，近期更新时间2017.12.27  

# 目录
更详细的目录等我找到好的auto generate table of content 再加上  
* [摘要](#摘要)
* [第一章 绪论](#第一章-绪论)  
* [第二章 统计参数语音合成基础技术综述)](#第二章-统计参数语音合成基础技术综述)  
* [第三章 中文语音合成系统的实现)](#第三章-中文语音合成系统的实现)  
* [第四章 个性化语音合成和情感语音合成的研究)](#第四章-个性化语音合成和情感语音合成的研究)  
* [第五章 总结和展望)](#第五章-总结和展望)  
* [第六章 附录)](#第六章-附录)  
    - [6.1 语音合成相关开源软件)](#6-1-语音合成相关开源软件)  
    - [6.2 语音合成的有趣应用)](#6-2-语音合成的有趣应用)  

# 摘要
随着语音识别技术的成熟，人们逐渐把研究焦点转向了语音合成系统。虽然语音合成已有多年的研究历史，但到目前为止，网络上尚未有基于统计参数语音合成的开源中文语音合成系统，也没有开源的中文语音合成语料库，而英文的语音合成所需的基础语料库和系统早已开源。随着语音合成音质的改善，人们对语音合成系统也提出了更高的要求，如要求个性化（特定人）的语音合成，情感语音合成，与此同时，人们也希望语音合成系统有更高的灵活性和实时性，能够利用有限的语音资源合成优质的语音。统计参数的语音合成系统因其对韵律的灵活控制，不需要很大规模的语料库成为了人们的选择，基于此，本文研究了基于统计参数的语音合成系统，同时综合利用最新的语音合成技术，在英文语音合成系统merlin的基础上搭建了一个开源的、具有灵活性、易于整合和修改的中文语音合成系统，以便今后的研究更好地开展。

本文首先对中文语音合成系统的文本处理前端进行了研究，主要包括了对文本规范化、中文分词、词性标注、拼音标注（包括对多音字的处理）、韵律结构预测的研究，其中我们着重研究了基于神经网络的韵律结构预测方法。之后，为了使系统具有更好的开放性和灵活性，本文研究并设计了上下文属性集、用于决策树聚类的问题集、分别基于音素（声韵母）和音节两种基础合成单元的标注文件。然后基于爱丁堡大学开源的英文语音合成系统merlin，设计并实现了基于HMM的中文统计参数语音合成系统。最后，我们针对当下的热点问题：个性化语音合成和情感语音合成又做了进一步的研究，探讨了如何在统计参数语音合成系统中通过韵律结构预测的改进和声学参数的变换来实现上述两个目标。

正文

# 第一章 绪论
## 1.1 研究背景
### 1.1.1 概述
### 1.1.2 自然语音模拟与合成
### 1.1.2 传统语音合成技术
### 1.1.3 深度学习在语音合成中的应用
最新的研究进展请参见[深度学习于语音合成研究-知乎](https://zhuanlan.zhihu.com/p/30776006)  
## 1.2 论文的研究内容
## 1.3 论文的结构安排
# 第二章 统计参数语音合成基础技术综述
## 2.1 汉语语言分析
TODO 这一部分见书《汉语自然语言处理》，等待补充
2.1.1 汉语拼音方案
中华人民共和国教育部发布的[《汉语拼音方案》](http://www.moe.edu.cn/s78/A19/yxs_left/moe_810/s230/195802/t19580201_186000.html)
## 2.2 隐马尔可夫模型
### 2.2.1 隐马尔可夫模型
### 2.2.2 HSMM半隐马尔可夫模型 
# 第三章 中文语音合成系统的实现
## 3.1 语料库
### 3.1.1 King_tts_003语料库
语料库使用购买自海天瑞声公司的King_tts_003语料库，该语料库由专业的标准女播音员录制，总时长为15小时，近2万个句子。其中标注包括了拼音、韵律（韵律边界与重音）、音素发音时长、声韵母标注。语音文件以44.1 KHz，16bit，双音道，windows的无压缩PCM格式存储。除了此外，该语料库还提供了记录EGG（electroglottography）信号的音频。

其中标记格式与含义为  
声调的标记格式  
采用数字1、2、3、4、5,代替《汉语拼音方案》中声调阴平（ˉ），阳平（ˊ），上声（ˇ），去声（ˋ），轻声（不标调）这几个标调符号  
   
**韵律的标记格式**
韵律分成四级，分别用#4，#3，#2， #1表示。   
#4  ：
（1）一个完整语意的句子，切除前后可以独立成为一个句子，从听感上调形是完全降下来的，有明显的停顿。   
（2）如果是以二声词结尾的短句，这个二声的词被拖长音，且与后面是转折的关系的，有明显的停顿。   
#3  ：
通常标在一个韵律短语后面，有时会是一个词，从听感上调形是降下来的，但不够完全，不能独立成为一个语意完整的句子。   
#2  ：
（1）表示被‘重读’的词或单个字(为了强调后面)，有停顿，调形上有小的变化, 有‘骤停’的感觉。 （对于单音节词如果是被‘拖长音’，给#1；如果是‘骤停’要给#2  ）
（2）并列关系的词如果被强调重读，给#2；如果是很平滑的，给#1。   
#1  ：
只是韵律词的边界，通常没有停  顿
   

**声韵母与停顿的标记格式**
标注符号采用a，b，d，s四种标记符号进行标注，标注符号的意思如下：
* a表示中文汉字的声母。
* b表示中文汉字的韵母。
* d表示句中的静音长度小于100ms的停顿。
* s表示句子的起始点和结束点以及句中大于100ms的停顿。

**声韵标注的具体规则**
1. 中文汉字拼音的声母用a表示，韵母用b表示。
2. 其中有一些汉字音节以元音开头，称为零声母音节，如a/o/e/ang/eng/en/ai/ei/ao/ou/an/er/，我们用标记点a来进行标注。
3. 其中有一些汉字是特殊读音，仅仅表示鼻子发出的气流，如m/n/ng/，分别对应汉字（呣，嗯，嗯），我们用标记点b来进行标注。
4. 汉字发音为yu/yi/wu/的为整体认读音节，但我们此次把以w，y为声母加韵母的拼音按照声韵进行切分。

举一个例子  
101001 我#1就怕#2自己的#1俗气#3亵渎了#2普者黑的#1风景  
wo3 jiu4 pa4 zi4 ji3 de5 su2 qi4 xie4 du2 le5 pu2 zhe3 hei1 de5 feng1 jing3  

## 3.2 文本分析
### 3.2.1 拼音标注规则
拼音标注风格分成两类，
1.第一类是国家规定的方案，也就是日常生活中用到的风格，具体参见中华人民共和国教育部发布的[《汉语拼音方案》](http://www.moe.edu.cn/s78/A19/yxs_left/moe_810/s230/195802/t19580201_186000.html)
2.第二类是方便系统处理的拼音标注风格，具体细分有很多种，这里使用的风格为：相比国家方案的其主要变化如下（建议的方法是将所有实际发音为v的改成v，将所有实际只有韵母的改成只有韵母的标记，例如yuan改成van，wan改成uan）
    * y w 如何处理，为了处理方便，是否将其当做声母，还是直接去除
    * 是否将ju qu xu 的标注改成 jv qv xu，注意到我们使用的语料库使用的是jv
        xv
    *
    注意到我们使用的语料库标注“援”这个音的时候使用的是yvan而不是yuan，而pypinyin中使用的是yuan
    *
    是否将i行韵母中出现的ye,yan,yang等改成ie,ian,iang，毕竟并不存在声母yw，同理于其他的uv行，注意到语料库使用的ye,yan,yang
    * 关于儿化音的处理？


## 3.3 文本分析工具包
常用的开源自然语言处理/开发包可见[知乎-鉴津Jackie的回答](https://www.zhihu.com/question/19929473/answer/264555333)

文本分析

下面将介绍几个笔者使用的自然语言处理工具包
### 3.2.1 结巴分词
结巴分词很容易安装和使用，可见[结巴分词-github主页](https://github.com/fxsjy/jieba)，这里就不赘述了。
值得注意的是
### 3.2.2 HanLp
### 3.2.3 NLTK
### 3.2.4 python-pinyin
[python-pinyin](https://github.com/mozillazg/python-pinyin)
提供了汉字转换成拼音的工具，它的安装和使用都很方便`pip3 install
pypinyin`，使用可见[官方文档](https://pypinyin.readthedocs.io/zh_CN/master/)  

关于拼音标注的风格，这里使用了pypinyin.Style.TONE3，举个例子`yuan2
jiu4`，再稍加转换就称为本系统中使用的风格

注意：
* pypinyin不提供对数字符号[0-9]进行拼音标记，使用时需要自定义词典，注意覆盖3.4中间有个“点”的声音



## 3.4 韵律处理
## 3.5 HMM训练
### 3.5.1 合成基元的选择
### 3.5.2上下文相关的标注
### 3.5.3 问题集的设计
参考微软论文:HMM-based Mandarin Singing Voice Synthesis Using Tailored Synthesis Units and Question Sets

**Question Set for Decision Trees**
Based on unit definition and contextual factors, we define five categories for the questions in the question set. The five categories of the question set are sub-syllable, syllable, phrase, song, and note. The details of the question set are described as follows.
1. Sub-syllable: (current sub-syllable, preceding one and two sub-syllables, and succeeding one and two sub-syllables) Initial/final, final with medial, long model, articulation category of the initial, and pronunciation category of the final
2. Syllable: The number of sub-syllables in a syllable and the position of the syllable in the note
3. Phrase: The number of sub-syllables/syllables in a phrase
4. Song: Average number of sub-syllables/syllables in each measure of the song and the number of phrases in this song
5. Note: The absolute/relative pitch of the note; the key, beat, and tempo of the note; the length of the note by syllable/0.1 second/thirty-second note; the position of the current note in the current measure by syllable/0.1 second/ thirty-second note; and the position of the current note in the current phrase syllable/0.1 second/thirty-second note 

### 3.5.4 决策树的构建
### 3.5.5 状态时长模型
### 3.5.6 基音周期模型
## 3.6 语音合成
语音合成的主要步骤有
1. 通过文本分析得到标注文件
2. 将标注文件转换为上下文相关基元的序列
3. 根据这个序列搜索得到相应的状态时长，基音周期和频谱的HMM模型
4. 由状态时长HMM模型得到基元个状态的持续时长
5. 根据状态的时长、基音周期HMM和频谱HMM，进行参数合成，得到每帧的基音周期、对数能量和、对数能量和MFCC参数
6. 将第5步得到的参数传入声码器进行语音合成
# 第四章 个性化语音合成和情感语音合成的研究
## 4.1 个性化语音合成文献综述
## 4.2 个性化语音合成研究
## 4.3 情感语音合成文献综述
## 4.4 情感语音合成文献综述
### 4.4.1 面向情感语音合成的上下文相关的标注格式
# 第五章 总结和展望
# 第六章 附录
## 6.1 语音合成相关软件
### 6.1.1 Praat
Praat语音学软件
http://www.fon.hum.uva.nl/praat/
### 6.1.2 HTK
### 6.1.3 HTS
### 6.1.4 Merlin
### 6.1.5 开源的自然语言处理项目及其比较
https://www.zhihu.com/question/19929473
目前常用的自然语言处理开源项目/开发包有哪些
## 6.2 语音合成的有趣应用
合成歌声 见https://www.zhihu.com/question/26165668)
参考文献
## 6.3 有用的工具
这里有与中文语言处理相关的Python package
https://pypi.python.org/pypi?:action=browse&show=all&c=98&c=489
