# Donchian Channel趋势跟踪策略设计
## 1 摘要
此研究报告主要参考Trading Systems and Methods, Chapter 8 Trend Systems中Bands and Channels提出的多种趋势判断方法，设计了一种趋势追踪策略。在进场方面，主要使用Donchian Channel搜索进场信号，利用Boll Band等多个指标进行信号过滤；在出场方面，利用CCI指标以及三级移动止损策略。研究以螺纹钢主力连续合约（R.CN.SHF.rb.0004）作为对象，数据频率为日线数据，单日限定交易一次，并以双向一手单的模式进行测试。
## 2 策略思路
## 2.1 核心指标含义及计算过程
1. Go long (and cover shorts) when the current price exceeds the highs of the previous four full calendar weeks.<br>
2. Sell short (and liquidate longs) when the current price falls below the lows of the previous four full calendar weeks.<br> 
3. When trading futures, roll forward if necessary into the next contract on the last day of the month preceding expiration.<br>
Donchian Channel趋势追踪系统的主要内容如上所述，该系统以某日的最高价突破通道上轨，或最低价突破通道下轨作为进场信号。Donchian Channel的上下轨计算公式如下：<br>
Upper_i=Max(H_(i-N),H_(i-N+1),H_(i-N+2)…H_i)<br>
Lower_i=Min(L_(i-N),L_(i-N+1),L_(i-N+2)…L_i)<br>
Upper_i：第i日的Donchian Channel上轨。<br>
Lower_i：第i日的Donchian Channel下轨。<br>
H_i：第i日的最高价。<br>
L_i：第i日的最低价。<br>
N：移动窗口大小。<br>

### 2.2 策略优化
Donchian Channel的作者Donchian在其基础上进行了如下改进：<br>
Donchian’s idea was to use a volatility-penetration criterion relative to the 20-day moving average, but with some added complication. The current price penetration must not only cross the 20-day moving average but also exceed any previous 1-day penetration of a closing price by at least one volatility measure. In this way Donchian places a ﬂexible band around the 20-day trendline. One volatility measure can be calculated as the average true range over one or more days.<br>
其主要内容是在价格突破轨道的同时，还需穿过移动平均线，并且满足波动率放大的条件。本研究在其基础上进行了部分优化，主要包括增加EMA优化趋势方向判断；使用VMW优化趋势强度判断；使用MACD优化趋势变化判断；使用Boll Width优化波动率判断；使用CCI过滤进出场位置；使用三级止损优化出场判断。<br>
### 2.2.1 趋势方向判断优化
考虑指数平均线相对简单平均线具有更灵敏的优势，本研究使用指数平均线代替原文中的简单平均线来反映趋势的方向。指数平均线的计算公式如下：
EMA_i=[C_i*2+EMA_(i-1)*(N-1)]/(N+1)
EMA_i：第i日的指数平均线。
C_i：第i日的收盘价。
N：移动窗口大小。
### 2.2.2 趋势强度判断优化
VMW反映了成交量对价格变动的影响，是衡量趋势强度的一种指标。其计算公式如下：
VMW_i=V_i*(C_i-C_(i-N))
VMW_i：第i日的成交动量。
V_i：第i日的成交量。
N：移动窗口大小。
### 2.2.3 趋势变化判断优化
MACD由快慢两组指数平均线构成，是衡量趋势变化的一种指标。MACD通常包含DIF,DEA以及MACD三个值组成，本研究仅使用其中的DIF值，其计算公式如下：
DIF=EMA(C,N)-EMA(C,M)
DIF：快慢线差值。
EMA：指数平均值。
N：快线窗口大小。
M：慢线窗口大小。
### 2.2.4 波动率判断优化
Boll Width是根据Boll Band中的上、下、中轨之间的关系计算得到的指标，反映了当前的波动大小。其计算公式如下
Mid=EMA(C,N)
Upper=Mid+2*STD(C,N)
Lower=Mid-2*STD(C,N)
Boll Width=(Upper-Lower)/Mid
Mid：轨道中轨。
Upper：轨道上轨。
Lower：轨道下轨。
N：Boll Band窗口大小。
Boll Width：Boll Band宽度。
### 2.2.5 进出场位置优化
Donchian Channel使用近期高点（低点）作为进场位置，因此使用Donchian Channel进行趋势追踪时，常出现在相对高位进场的情况，从而容易出现较大的回撤，本文使用CCI（顺势指标）来对趋势的相对位置进行判断，过滤入场时机较差的信号。其计算公式如下：
TP=(H+L+C)/3
MA=(∑C_i )/N
MD=(∑MA_i-C_i)/N
CCI=(TP-MA)/(MD*0.015)
TP：K线中值。
MA：简单移动平均。
MD：简单移动平均差。
N：窗口大小。
CCI：顺势指标。
### 2.2.6 出场位置优化
尽管Donchian Channel能够较好地发出进场信号，但缺少发出出场信号的能力，本研究在CCI的基础上，设计三级止损方法，用于计算出场点位。三级止损共包括三个止损线，当最低价（最高价）触发其中任意一条时出场，三条止损线计算公式如下：
Stop_1=E*(1-D*R_1)
Stop_2=E*(1+D*R_2)
Stop_3=HE or LE*(1-D*R_3)
E：建仓价。
D：建仓方向，多仓时取1，空仓时取-1。
Stop_1：一级止损，即绝对止损。
R_1：绝对止损比例。
Stop_2：二级止损，即保本止损。
R_2：保本止损比例。
Stop_3：三级止损，即追踪止损。
R_3：追踪止损比例。
