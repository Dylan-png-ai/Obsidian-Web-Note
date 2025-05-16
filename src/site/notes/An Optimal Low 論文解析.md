---
{"dg-publish":true,"permalink":"/An Optimal Low 論文解析/"}
---


### 想解決的問題?
解決OFDM系統特有的問題:「高PAPR」。因為將訊號直接做ifft形成時域的調變訊號，在某些時刻上可能導致訊號疊加，造成高的峰值，使 Peak-to-Average Power Ration 很高。這樣會導致[[PA 操作在非線性區導致的訊號表現\|發射機PA操作在非線性區]]，讓系統不太理想。因此使用一些技術降低PAPR在multicarrrier系統上是重要的。

### 目前已有的研究?
Distorsionless: coding、PTS、GPW、SLM、Tone Resevation、ACE
Distorsion: clippling、peaking windowing、peak cnacellation
其他: LINC(不從訊號下手，從功率放大器改善)
![wcsp_ch3_PAPRtech.png](/img/user/attachments/wcsp_ch3_PAPRtech.png)

### 提出甚麼方法?
- 發射端測量 OFDM modulator 的輸出 ( IFFT 後時域訊號的係數)，將超過閾值 a 的所有係數乘上一個倍數 b (b介於0~1之間 讓訊號強度縮小)，倍數 b 以及被縮放過的係數索引會以另外的 side information 傳送至接收端。
- 接收端在 OFDM demodulator 之前 ( FFT 前)，依據收到的索引將訊號除上倍數 b，接下去就是一樣的系統。
- 方法的目標: maximize achievable PAPR reduction and a controllable level of distorsion

### 最後的成效如何?
##### 主要的優點:
1. 有效的降低 OFDM 系統的 PAPR。
2. 額外處理訊號的演算法在發射端是線性的；在接收端是次線性的(低於線性)。
3. 主要的 trade off 就是，要頻譜利用率高能傳送的 side information 不能太多，能提供的 PAPR 成效較低；反之亦然，同時BER在各狀況都能維持好的表現(與不做reduction系統相當)。
##### 次要的優點:
1. 提出了Net Gain的概念，通常 PAPR 有好的表現就會讓訊號 BER 變差(訊號失真嚴重)，因此這篇 paper 將 PAPR 所帶來的效益扣掉訊號失真的影響，以全面的角度審視一個 PAPR reduction 技術為系統帶來的淨增益。
2. 成果發現(也可以由公式(38)知道) PAPR值跟OFDM所用的子載波數只有些微的關聯性，也就是當子載波數增加時PAPR的表現依然好。因此這套系統可以用在未來擁有大量子載波數的通訊標準。這個技術歸類在 distorsion，但是他也有傳送 side information，這跟 distorsionless 的技術一樣需要側訊息，但不同之處在於 distorsionless 技術對側訊息的完整度非常敏感，因此都要對側訊息做多層的 channel coding，進一步降低頻譜效率(系統的；這個技術的側訊息不需要多做保護，在嘈雜的通道中依然可以維持不錯的 BER，更簡單卻更robust。
### 與其他技術相比呢?
- 黑線是本篇paper在不同頻譜利用率下的CCDF表現，可以看到只有Peak clipping比他好。但是peak clipping會引入嚴重的失真(BER表現不好)。![figure9.CCDF比較圖.png|500](/img/user/attachments/figure9.CCDF%E6%AF%94%E8%BC%83%E5%9C%96.png)

- 比較了distorsion技術間的BER表現，paper的技術都是貼著未經處理的原始曲線，BER的表現優。![figure11.BER比較圖.png|500](/img/user/attachments/figure11.BER%E6%AF%94%E8%BC%83%E5%9C%96.png)
這張 BER 沒有畫到 distorsionless 技術的表現，因為前提假設是通道都完美，看系統對 BER 的損害。distorsionless 技術在通道完美的情況下，一定可以完美還原出原本的訊號。

- 複雜度(發射端)比較:
	本文技術：O(N)->線性
	**Clipping and Filtering**：O(NL)，其中L是FIR濾波器的長度->線性
	**Selective Mapping (SLM)**：O(SN log2​N)，其中S是相位序列的數量->超線性
	**Partial Transmit Sequence (PTS)**：O(NP^2)，其中P是partial序列的數量->超線性

### 系統模型:
![system model.png](/img/user/attachments/system%20model.png)

### 問題集:

##### **公式(3)中的 x[i] 也就是ifft後的係數，為何可以假設為 "iid zero-mean complex Gaussian random variables" (在N大等於256時假設成立)? paper page 16410.**
> 1. 這個會成立的前提是做ifft的輸入本身就是 iid zero-mean (ex. From QAM symbol)。因此 ifft 只是將多個調變的子載波加總起來，也就是將這些 r.v. 加起來，其統計特性並沒有改變。
> 2. 成立的另一個條件是 Central Limit Theorem 告訴我們，無論原始 r.v. 個別分佈如何，大量獨立的 r.v. 相加起來的分布會趨近於高斯分佈，因此論文提到子載波數(N)必須大於256個

##### **Why is |x[i]| Rayleigh-distributed with uniform phase? paper page 16410.**
> 1. 首先要知道 statistic result，當 a 與 b 是兩個獨立的隨機變數，並且同時 **zero-mean、same variance、都是高斯分佈**，則一個負數隨機變數 c=a+bj 的強度 |c|=sqrt(a^2+b^2) 會遵守 Rayleigh Distribution，distribution 的 var 會跟 a 和 b 的 var 相同
> 2. 這裡的 phase 指的是複數在複數平面上的角度 θ=arctan(a/​b​​)，這個 phase 會是均勻分布在0~2π之間的，概念上來說高斯分佈在複平面上具有旋轉對稱性（無方向偏好），相位 θ 的分佈自然均勻。正式的[[Rayleigh distribution 推導.png|數學推導]]

##### **PAPR reduction可以讓傳送端的PA操作在更好的位置，這樣一來就可以提升TSNR，為什麼? paper page 16414.**
>1. OFDM調變的訊號具有高PAPR的特性，也就是訊號有大的動態範圍，這樣在發射訊號時可能會導致 Power Amplifier 進入非線性操作區，導致[[PA 操作在非線性區導致的訊號表現\|訊號失真或out-of-band emission]]。因此發射端的PA通常都會做一個 power back-off的機制，也就是讓PA的操作點退離最佳操作點多一點，以此來克服有很大的peak迫使PA進入非線性區，讓所有訊號都能在線性區中被放大。
>2. 但power back-off的缺點就是會讓PA的效率很差，退離最佳操作點會導致訊號被放大的程度降低，訊號功率不夠大，導致訊號進入通道後被雜訊埋沒，TSNR就下降了。
>3. 所以PAPR Reduction可以讓訊號的PAPR更理想，這樣PA就不需要很多的power back-off去確保工作在線性區，讓PA有效的把訊號功率提升，接收端就能得到更高品質的訊號。

##### **Figure 6(a) 中，以Ms=64的情況下，為什麼曲線是隨TSNR增加，Net Gain先下降在上升? paper page 16414.**
> 1. 首先要知道，64QAM的錯誤率曲線是隨著SNR指數性下降的，也就是在SNR低的時候，錯誤率下降的速度很慢(斜率小)；在SNR高的時候，錯誤率下降的速度很快(斜率大)
> 2. 注意G可以由(41)得到，觀察公式(41)，分子ρxλ在選定Ms後就固定了。會隨TSNR變動的項目剩下K，K在分母，而K中有TSNR項和Pw項相乘。對於64QAM來說，低TSNR時Pw的值也很大(錯誤率高)，即使TSNR增加使Pw的改善(要變小)也不明顯，TSNR會主導K。因此TSNR上升，K值變大，K值變大導致G下降(K在分母)
> 3. 到了TSNR足夠高時(>15dB)，64QAM的錯誤率Pw會顯著改善(變很小)，因此Pw主導K，K也會變很小。因此TSNR上升，K值變小導致G上升(K在分母)。[[figure 6(a) Ms=16 曲線下凹解釋\|詳細說明]]

##### **為什麼公式(24)是"the worst word error"**
> 有兩個前提假設
> 	1. 每個binary code word(二進位的indices)的錯誤是獨立的
> 	2. 在最壞的情況下，只要有一個 bit 錯誤則整個binary code word就錯了
> 
> 每個indices需要log2(N)個bit來表示，每個調變符號可以攜帶的是log2(Ms)個bit，因此傳送一個indice所需的調變符號數量是log2(N)/log2(Ms)，根據公式(23)，每個調變符號的錯誤率是Ps。組合再一起就代表調變符號的數量乘上每個符號的錯誤率，也就是這個indice(binary code word)會在接收端被解錯的機率Pw，由公式(24)定義。

##### **Side information的編碼並傳輸的機制不太明確很多問號???**
> 1. Side information 有提到是用binary encoded再做MQAM mapping，這邊具體怎麼encode不知道，把b和indices個別轉成二進位後再做MQAM嗎?
> 2. 有提到系統頻寬必須犧牲一些給side information的傳遞，而side information是完全獨立傳送的，怎麼同步獨立傳送? 這樣要怎麼知道哪一個OFDM symbol他的scaling factor和indices?
> 3. 考慮系統複雜度的時候為什麼都沒有提到處理side information的運算複雜度? 
> 4. side informaiton的傳送速率該怎麼定義? symbol duration是多少? 跟OFDM一樣嗎?
> 5. 沒有分析如果side information錯在傳送scaling factor b上的時候，對系統的影響
> 6. 每一個OFDM symbol都需要去傳送相應的side information，怎麼同步?
> 7. RSNR的分母是訊號和重建訊號的MSE，為什麼? 這樣定義合理嗎?

##### 報完老師的 feedback
> 1. 這篇論文的模型在考慮通道上有點太理想了，所以結果呈現出來很好，但實際的通訊環境下是否仍有如此好的結果需要進一步求證
> 2. 考慮到更實際的層面，論文想要解決PA進入非線性區的問題，但到底針對哪一個PA沒有提到，PA操作的工作點在哪也沒有說。在實際應用的時候，這些也都會左右一套方法的結果和設計方式。

##### 延伸題目
> 老師有針對這個題目給一些建議，去嘗試間隔擺放子載波，例如原本48點的訊號做64點的IFFT，原本有48個subcarrier，但是現在把48拆成4份，一份有12，這樣只在頻譜上擺12隻subcarrier，同樣去做64點的IFFT，實際的時域訊號會有64點，==但應該出現四個相同的block?== 所以可以取前四分之一(16點)作為有用的訊號，這樣一來，雖然犧牲了傳輸的rate(48/4=12，一次只傳12個)，但一個OFDM symbol的長度卻下降了，長度下降帶來的好處有較不受CFO影響、==把四個16點的時域結果拼起來PAPR也有影響?== 用這篇提到的一些概念和分析技巧，去看和比較 trade off 也許有不錯的收穫。可不可行，需要實作和找也許有論文做過相關的問題。
> ![老師手寫.png](/img/user/attachments/%E8%80%81%E5%B8%AB%E6%89%8B%E5%AF%AB.png)