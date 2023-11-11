---
tags:
  - QR
---
# Abstract

QR là viết tắt của quick response -> mục đích đầu tiên để nó được tạo ra là để track parts across the manufacturing process.

Nó đồng thời được dùng để khắc phục 1 số điểm hạn chế của barcode như:
- Barcode chỉ có thể đọc được từ 1 hướng, QR code có thể được đọc từ mọi angle
- QR có thể lưu nhiều thông tin hơn do nó là dạng ma trận 2 chiều còn barcode là dạng 1 chiều (tối đa 20 kí tự)

## Error Corection Level
QR codes contain error correction data. The standard offers the following levels of error correction:
- L: (low) about 7% of codewords can be restored
- M: (medium) 15%
- Q: (quality) 25%
- H: (high) 30% (not available for Micro QR codes)

For Micro QR code symbols, the available error correction levels depend on the version:

- M1 has only error detection
- M2 and M3 support L and M levels
- M4 supports L, M and Q levels

# Technical detail

Các mã QR thường có định dạng: các ô hình vuông màu đen được xắp sếp trên nền trắng

> A QR code is a square array of dark and light dots. One dot (or "module") represents one bit of information.

## QR Code Version

Có đến 40 version khác nhau của QR code. Mỗi version khác nhau chủ yếu về kích thước của QR code. Version 40 có kích size là 177 \* 177

![[Pasted image 20231108182153.png]]

Mỗi version đều khác nhau về các yếu tố:
- maximum data capacity
- error corection level
- character type

!! Image ->



QR code được chia làm nhiều loại với các kích thước khác nhau. Tuy nhiên mọi QR code đều bao gồm các thành phần sau:

1. **Quiet Zone**
2. **Separator**
3. **Finder**
4. **Aligment marking**
5. **Timing**
6. **Version Information**
7. **Format Information**
8. **Content**

![[Pasted image 20231108014847.png]]


Các thành phần không lưu trữ data chính bao gồm: finder parttern, separator, timing pattern và alignment patterns được gọi là function pattern
## Finder

Điểm nổi bật nhất trong các mã QR chính là 3 hình vuông có kích thước lớn nằm ở 3 góc của mã QR

![[Pasted image 20231109011243.png]]

Đây là các vị trí để dánh dấu hướng đọc của mã QR -> nó giúp các ứng dụng có thể dọc mã QR ở bất kỳ angle nào. Hình vuông nhỏ bên trong gọi là **Inner eye**, hình vuông bên ngoài gọi là **Outer eye**.

Hình vuông phía bên ngoài mà đen (Outer eye) có kích thước là: 7 * 7, trong nó là hình vuông màu trắng có kích thước 5 * 5, và hình vuông màu đen trong cùng(Inner eye) có kích thước 3 * 3. -> Các dải hình vuông bên ngoài có độ rộng là 2.

Khoảng trắng ở giữa finder pattern và phần lưu trữ dữ liệu được gọi là separator.

## Timing pattern

Timing parttern giúp các ứng dụng biết được độ lớn - kích thước của mã QR -> tức là chỉ ra được mã QR này là version bao nhiêu.

Tiiming parttern được detect dựa vào finder như hình dưới

![[Pasted image 20231110011326.png]]

Chúng gồm 2 dải các ô trắng đen liên tiếp (như hình vẽ) được gọi là vertical timing parttern và  horizontal timing pattern. 

- Horizontal timing pattern nằm ở cột thứ 6 (từ trái sang)
- Vertical timing pattern nằm ở dòng thứ 6 (từ trên xuống)

## Alignment pattern

Alignment pattern có tác dụng giúp các phần mềm scan mã QR xác định được 

Kích thước của alignment patter từ ngoài vào trong lần lượt là 5 * 5 3 * 3 và 1 * 1

## Format info

Format Info cũng được lưu ở gần các finder.

![[Pasted image 20231110011737.png]]

Từ phần format information này ta có thể biết được:
- Mask
- Error Correction level
- Error Correction format

# Error Corection

Để có được khả năng error corection, data trong QR Code được lưu redundant

![[Pasted image 20231112002045.png]]

QR Code sử dụng thuật toán [[Reed-Solomon]] để thực hiện việc sửa lỗi hay khôi phục dữ liệu trong trường hợp 1 phần của mã QR bị hư hại, không thể scan được.

# QR Generation process
1. Analysis Input data
2. Data Encoding
3. Error corection coding using Reed-Solomon Algorithm
4. Structure final message
5. Module placement
6. Perform data masking
7. Add format and Version Information
8. Output QR Code
# QR Code scanning process

1. Point your phone at a QR code.
2. The QR code scanner in your phone’s camera recognizes the three position markers in the QR code. With a sufficient quiet area, your scanner is now aware of where the edges of the QR code are.
3. The scanner begins at the bottom right, where it encounters the _mode indicator._ These four data modules indicate what data type (numeric, alphanumeric, byte, or kanji) the rest of the encoded data is.
4. Next, the scanner encounters the _character count indicator,_ which are the next 8 data modules up from the mode indicator. These indicate how many characters the total encoded data is
5. Knowing the data type and character length, the scanner then continues its zig-zag path along the data modules until all it retrieves all the encoded information and reaches the _end indicator_
6. After reading the final character, the scanner proceeds along its path to the error correction data modules. Within these encoded modules are one of four levels of error correction. Or how much of the QR code’s encoded data is backed up in case of code damage

![[Pasted image 20231109012554.png]]

# Type of QR Code

## QR Code Model 1

QR Code model là tiêu chuẩn (Specification) đầu tiên của mã QR, nó được giới thiệu vào năm 1994.

Phiên bản đầu tiên này sử dụng ma trận có kích thước tối đa 73 * 73 để lưu trữ. Nó tương đương với phiên bản 14 của Version 2 -> Có nhiều phiên bản, nhưng tối đa là Version 14. Điều đáng chú ý là QR Model 1 chỉ lưu trữ được numeric data only.

Loại mã QR này hiện ít được sử dụng do sự ra đời của Version 2 với nhiều cải tiến hơn.
## QR Code Model 2

Là loại mã tiêu chuẩn ở thời điểm hiện tại ([[2023-11-11]]). Nó gia tăng số lượng version lên 40 (tương đương với việc đưa kích thước của mã QR lên 177 * 177). -> Điều này cũng kéo theo việc lưu trữ được nhiều dữ liệu hơn và khả năng sửa lỗi cũng đượch tăng lên.

Ngoài ra nó cũng có thay đổi khả năng lưu trữ, không chỉ là lưu được các dữ liệu numeric mà còn có thể lưu được chữ, binary data -> khả năng ứng dụng cao hơn.
## Micro QR Code

aka Small QR Code

Nó nhỏ hơn so với mã QR thông thường (Hiển nhiên small mà 🥲🥲) -> Phù hợp với các trường hợp mà chỗ in mã QR có diện tích nhỏ (như trên thiết bị vi tính). và cũng bởi vì size của chúng nhỏ hơn mã QR thường nên lượng data mà chúng chứa được cũng ít hơn. Và kèm theo đó là khả năng phục hồi lỗi cũng thấp hơn.
Micro QR có 4 version.

| Version | Modules | Maximum alpha numeric characters held |
| ------- | ------- | ------------------------------------- |
| M1      | 11 * 11 | None, only 5 numeric characters       |
| M2      | 13 * 13 | 6                                     |
| M3      | 15 * 15 | 14                                    |
| M4      | 17 * 17 | 21                                      |

Ưu điểm của Micro QR là:
- Nó chỉ cần 1 Finder pattern
- Chỉ cẩn 1 quiet zone
## iQR

iQR QR Code được coi như là 1 phiên bản mở rộng, nâng cấp của mã QR tiêu chuẩn. Tuy nhiên loại mã này được độc quyền sử dụng bởi Denso Wave (công ty sáng tạo ra QR Code). iQR có nhiều ưu điểm hơn so với các mã QR tiêu chuẩn thông thường như:
- Có thể store được nhiều data hơn so với QR thường
- iQR có thể được biểu diễn dưới dạng hình chữ nhật chứ không bắt buộc là hình vuông
- Tăng khả năng tự sửa lỗi

## SQRC hay EQR

aka Encrypted QR Code - là 1 loại QR mà dữ liệu được mã hóa. Điều này làm cho thông tin trên mã QR có thể được chia sẻ một cách an toàn và bí mật.

## Frame QR

Loại mã QR này sẽ cho phép chèn hình ảnh hoặc các dạng đồ họa khác vào chính giữa của mã QR. Vùng đồ họa này trên mã QR được gọi là QR Code frame

# References

1. [QR codes | Dan Hollick (typefully.com)](https://typefully.com/DanHollick/qr-codes-T7tLlNi)
2. [QR code - Wikipedia](https://en.wikipedia.org/wiki/QR_code)
3. [QR Code Types: Different Types of QR Codes (sproutqr.com)](https://www.sproutqr.com/blog/qr-code-types)
4. [(PDF) An Introduction to QR Code Technology (researchgate.net)](https://www.researchgate.net/publication/318125149_An_Introduction_to_QR_Code_Technology)