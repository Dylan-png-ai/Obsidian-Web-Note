---
{"dg-publish":true,"permalink":"/An Optimal Low Complexity PAPR Reduction Technique for Next Generation OFDM Systems/"}
---


### 想解決的問題?

### 提出甚麼方法?
- 發射端測量 OFDM modulator 的輸出 ( IFFT 後時域訊號的係數)，將超過閾值 a 的所有係數乘上一個倍數 b (讓訊號強度縮小)，倍數 b 以及被縮放過的係數索引會以另外的 side information 傳送至接收端。
- 接收端在 OFDM demodulator 之前 ( FFT 前)，依據收到的索引將訊號除上倍數 b，接下去就是一樣的系統。
- 方法的目標: maximize achievable PAPR reduction and a controllable level of distorsion

### 最後的成效如何?
### 與其他技術相比呢?

### 系統模型:
![system model.png](/img/user/attachments/system%20model.png)

### 問題集: