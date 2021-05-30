# Phụ thuộc
Python 3.6

- Tensorflow-gpu 1.2.1
- Numpy
- Soundfile
- PyWorld
  - Cython
<br/>


# Cài đặt môi trường
- Tạo thư mục chưa project (ví dụ `dsc`)
- cd đên folder `dsc`
- Tạo môi trường ảo: `python3 -m venv venv`
- Cài đặt môi trường cần thiêt:
  `pip install -r requirement.txt`

# Tiền xử lý dữ liệu
- Chuyển đổi âm thanh được ghi âm (origin) định dạng m4a/mp3 sang wav, mono chanel + sr = 16kHz

<br/>

# Sử dụng
1. Run `analyzer.py`  để trích xuất đặc trưng và lưu các đặc trưng đó thành các file nhị phân (*.bin).
2. Run `build.py` để lưu một số thông kê, chẳng hạn như cực trị phổ và pitch.
3. Để huấn luyện mô hình, Run 
```bash
python main.py --model ConvVAE --trainer VAETrainer --architecture architecture.json
```
4. Model sau đó được lưu vào địa chỉ: `./logdir/train/[timestamp]`  
5. Để chuyển đổi giọng nói, Run
```
python convert.py \
--src SM1 \
--trg TM1 \
--model ConvVAE \
--checkpoint logdir/train/[timestamp]/[model.ckpt-[id]] \
--file_pattern "./dataset/bin/Testing Set/{}/*.bin"
```  
*Điền đủ tham số  `timestampe` và `model id`.  
5. Các file được chuyển đổi sau đó được lưu vào địa chỉ: `./logdir/output/[timestamp]`  

<br/>

# Bộ dữ liệu
- Bộ dữ liệu được ghi âm bởi trình ghi âm của điện thoại Redmi Note 8 + Sol Prime T1000 + SS A8 Star.
- Gồm có 4 speaker:
    - {`SM1` - `Giọng tôi - PXP`}
    - {`SM2` - `Giọng anh tôi - PXĐ`} 
    - {`TM1` - `giọng nam trung`} 
    - {`TM2` - `giọng nam bắc`}
- Ghi âm 120 utterances / 1 người. Tổng cộng có 720 utterances
<br/>

# Mô hình  
- Variational Autoencoder (VAE)
- Lossfuntion: 
    - reconstruction loss (MSE/cross-entropy)
    - Kullback Leibler divergence loss
- Metrics:
    - Mel Cepstal Distortion (MCD)
    - MOS score
    - speaker similarity
# Cấu trúc file/folders
```
dataset
  bin
  wav
    Training Set
    Testing Set
      PXP
      PXD
      TBT
      PDH
etc
  speakers.tsv
  (xmax.npf)  
  (xmin.npf)  
util (submodule)
model
logdir
architecture*.json

analyzer.py    (feature extraction)
build.py       (stats collecting)
trainer*.py
main.py        (main script)
(validate.py)  (output converted spectrogram) 
convert.py     (conversion)
```
<br/>

# Định đạng dữ liệu nhị phân
Các đặc trưng của WORLD vocoder và nhãn (label) của người phát âm được lưu trữ ở định dạng nhị phân.
<br/> 
Format:
```
[[s1, s2, ..., s513, a1, ..., a513, f0, en, spk],
 [s1, s2, ..., s513, a1, ..., a513, f0, en, spk],
 ...,
 [s1, s2, ..., s513, a1, ..., a513, f0, en, spk]]
```
Trong đó:  
`s_i` is spectral envelop magnitude (in log10) of the ith frequency bin,  
`a_i` đặc trưng `aperiodicity` tương ứng,    
`f0` là tần số cơ bản/pitch (0 cho các frame không có âm thanh) 
`en` là năng lượng 
`spk` là chỉ số của người phát âm (0-3) và  `s` là `sp`.
<br/>

# Chú ý:
Nếu sử dụng Tensorflow-CPU, bạn thay thế tất cả toán tử `NCHW` trong source code thành `NHWC`