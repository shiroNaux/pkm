---
---
# Abstract

Data vault là 1 pháp data modeling. Nó được phát triển sau [[Dimensional Modeling]] nên nó hạn chế được 1 số nhược điểm của Dimensional Modeling.

# Architecture

Ở đây, chúng ta sẽ nói đến Data Vault 2.0, là phiên bản được sử dụng ở thời điểm hiện tại (([[2023-08-23]]))

- Data vault modeling là 1 thành phần của Data Vault

Các bảng trong Data Vault Modeling được chia ra làm 3 loại chính
- Hub -> Các bussiness __entities__ được quan tâm
- Link -> Liên kết/Link giữa các Hub với nhau.
- Satellites -> Là bảng chứa các thông tin chi tiết, bổ sung cho Hub và Link

## Core concepts

Các thành phần chính của Data vault modeling bao gồm 3 đối tượng
- Hubs
- Links
- Satellites

![[Pasted image 20230823011151.png]]

Ngoài ra, để phù hợp với nhiều yêu cầu đặc thù, một số loại bảng hỗ trợ được thêm vào, cùng với đó là 
### Hubs

> Hubs are entities of interest to the business.

Hubs là các __đối tượng business__ được quan tâm. Các bảng Hubs chỉ chứa Business key của các object để thể hiện là đối tượng này có tồn tại trong hệ thống. Các thông tin metadata về đối tượng được lưu trong bảng satellites.

Hubs có ý nghĩa như là 1 bảng ghi lại sự xuất hiện của đối tượng nào đó trong hệ thống.Vì thế các bảng Hubs sẽ chỉ chứa business unique key của đối tượng (Business Key này có thể là composite fields).

Tuy nhiên, do dữ liệu có thể xuất phát từ nhiều nguồn khác nhau, cho nên các bảng Hub cần có trường Data source để cho biết data này tới từ nguồn nào.
Hub cũng sẽ Surrogate Key như Dimensional Modeling -> phân biệt các đối tượng có thể tới từ các nguồn khác nhau. Giá trị của Surrogate Key này thường sẽ là [[Hash function|hash]] value của: Business Key + Source

Ngoài ra, các bảng Hub có thể có thêm 1 số cột metadata khác như load timestamp, ...

Có thể thấy là cấu trúc của các bảng Hub khá đơn giản, nó chỉ chứa Business Key của hệ thống nguồn và 1 số trường hỗ trợ cho quá trình xử lý dữ liệu
### Links

>  Links connect Hubs and may record a transaction, composition, or other type of relationship between hubs

Links là các relationship giữa các Hub -> many to many join tables. Và đương nhiên nó có thể là quan hệ giữa nhiều bảng với nhau. Nó chính là đại diện cho mối quan hệ giữa các đối tượng (Hub) trong thực tế.

Một bảng Links tối thiểu cần có:
- SUROGATE KEY của các Hubs mà Links là liên kết giữa chúng
- SUROGATE KEY của Links
- Các cột hỗ trợ cho việc xử lý dữ liệu: load_ts, source ...

Tất nhiên là Links cũng sẽ có Surrogate Key và các cột hỗ trợ việc xử lý khác như load timestamp, Record source, ...
#### Hierarchical Link

Là các Links được dùng để diễn tả mối quan hệ của các đối tượng trong cùng 1 Hub. Ví dụ: Links của cùng Hub mô tả quan hệ cha con giữa các địa điểm trong HUB_AREA
#### Same-as Link

Là quan hệ giữa cùng 1 đối tượng. 
Ví dụ: Trong bảng Hub có 2 giá trị SĐT, 1 giá trị sai và 1 giá trị là đã được chuẩn hóa -> Khi đó ta có thể dùng 1 Links để liên kết 2 số ĐT này để cho biết đâu là giá trị chính xác

#### Non-historized Link / Transaction Link

> That is perfectly adapted to the immutable facts, so something that happened in the past can neither be undone nor modified. And since the changes will never happen, they don't need to be stored in the satellites

-> Tức là các Link này chứa data không thể bị update (Log, Transaction, ...). Khi đó dữ liệu sẽ được lưu trực tiếp trong bảng Link, không cần bảng satellite -> Join ít hơn.
#### Exploration Link

Đây là 1 loại Link phục vụ cho Business Rules.
Ví dụ: 
- Khi ta muốn biết rằng User có thích Product nào hay ko, ta hoàn toàn có thể tạo thêm 1 bảng Link giữa User và Product. Bảng này giống như 1 model thống kê, đo sự tương quan giữa User và Product để biết là họ có thích sản phẩm này hay không.

Việc thêm, hay xóa cũng như thay đổi trên các bảng loại này không anh hưởng đến các thành phần khác trong Data Vault.
### Satellites

> 

Satellite được coi là `context` của các đối tượng khác (Hubs, Links). Nó cung cấp các descritive data cho các đối tượng đó. Ví dụ: _Satellite_ của _Hub_User_ có thể sẽ có các thông tin về Tên , tuổi, địa chỉ, ... của User đó

Các Hubs hoặc Links có thể có nhiều hơn 1 satellite -> Điều này nhằm hướng tới khả năng mở rộng cũng như có thể dùng để tăng hiệu năng của các truy vấn (gần giống với sharding).

Không có điều ngược lại. Các Satellite chỉ có 1 liên kết với các bảng Hubs hay Links. Giữa các Satellite cũng không có direct relationship.
#### Multi-active satellite

Các bảng satellite này chứa nhiều giá trị active active/valid at a time.
Ví dụ: 1 người có thể có nhiều số ĐT -> Satellite chứa Phone Number chứa nhiều giá trị active này

Đối với các bảng này, có cách model:
	- Row-based
	- Set-based
#### Efective satellite

Là các Satellite chứa các records mà chỉ có giá trị trong 1 khoảng thời gian nhất định -> Các bảng satellite này thường có thêm cột END_VALID_TIME và START_VALID_TIME

#### Business satellite

Là các bảng giống như satellite, nhưng được sinh ra trong quá trình Transform, Enrich data. Thường là các bảng aggregated -> đưa ra kết quả nhanh hơn, phục vụ cho business
### Additional Table

#### Point-in-time table (PIT)

Các bảng PIT và Bridge được coi là _dispoable_ -> có thể phá hủy, bản chất của các bảng này chỉ là được delivered từ các bảng có sẵn.

Tác dụng chính của các bảng này là giảm thiểu độ phức tạp của query và tăng performance.

PIT Tables được sử dụng trong trường hợp: 
- Khi mà 1 Hub có nhiều Satellites. Và ta cần query từ tất cả các satellite đó
- Ví dụ như ảnh dưới

![[Pasted image 20230911013853.png]]
Nếu sử dụng các bảng trong câu [[SQL|truy vấn]] thì sẽ phải sử dụng 1 đoạn query rất dài và phức tạp vì
- Join nhiều bảng
- Các bảng satellite có chứa state (trạng thái) -> phải có điều kiện where rất phức tạp để lấy đúng được các giá trị khớp với nhau từ tất cả các bảng satellite

Và ta cũng không thể kết hợp tất cả các bảng này vào trong 1 bảng satellite duy nhất bởi nhiều lí do (Data từ các nguồn khác nhau, change rate khác nhau -> tách bảng ra để tăng perfomance)

Để giải quyết vấn đề trên, các bảng PIT được tạo ra theo cách: 

For example, if a hub has 3 different satellites, satA, satB and satC, a point-in-time table will store the most recent load date for every satellite and so for every business key. So the stored data will be built like (business key [BK] from the hub, MAX(loadDate from satA for BK), MAX(loadDate from satB for the BK),MAX(loadDate from satC for BK)

Từ Model ban đầu, khi ta sử dụng PIT thì sẽ có được diagram như sau
![[Pasted image 20230911014818.png]]

#### Bridge table

Đây cũng là 1 loại bảng dùng để tăng performance cho query. Dữ liệu trong Bridge tables cũng được delivered từ các bảng khác.

Do kiến trúc của DV, nên các câu truy vấn phải thực hiện join rất nhiều -> Phức tạp và slow. Để giải quyết vấn đề này, người ta đã sử dụng Bridge tables (BRG). Các bảng này giống như wide table, chứa hầu hết các thông tin cần thiết vào trong 1 bảng duy nhất để tránh truy vấn trên nhiều bảng. Nghe có vẻ giống PIT, nhưng điểm khác biệt ở đây là PIT thì chỉ lưu data của 1 Hub duy nhất, còn bridge thì có thể lưu dữ liệu của nhiều bảng khác nhau

Cấu trúc của 1 Bridge table thường nhưu sau:

![[Pasted image 20230912013531.png]]

Ví dụ: Đối với các table Location, nếu ta dùng DV để model thì kết quả thường sẽ giống thế này
![[Pasted image 20230912012224.png]]

Trong trường hợp này, ta có thể áp dụng BRG table như sau

![[Pasted image 20230912012314.png]]


#### Reference table

Là các bảng thường được reference nhiều bởi các bảng khác

Ví dụ: Satellite có thể chứa Category Code để reference đến bảng Category mà không cần phải tạo thêm bảng Link giữa Product và Category -> giảm sự phức tạp ko cần thiết

Thường thì các bảng reference sẽ là 1 Hub, và các bảng satellite sẽ có Foreign Key tới bảng này.
## Layer

Thông thường các Data warehouse mà implement Data vault method cũng sẽ chia ra làm 3 layers giống như Dimensional Modeling
# Modeling steps

Có nhiều cách tiếp cận với 

Sau khi thiết kế được DV với các bảng tiêu chuẩn: Hubs, Links, Satellites thì coi như đã hoàn thành được bước modeling. Tuy nhiên, vẫn cần 1 số bước tạo ra các bảng hỗ trợ như: refernce tables, PIT, Bridge (BRG), .. để có thể khiến DV hoạt động 1 cách hiệu quả

## Rules

This is some rules we should follow when modeling with Data Vault 2.0

![[DV-datasheet-v109-A3.pdf]]

# Data Warehouse Architecture

Below is a proposal for Enterprise Data warehouse Architecture
![[Pasted image 20230906010912.png]]

# Comparation

## Pros
- Insert only architecture: Cái này sẽ phù hợp các hệ thống data warehouse hiện đại, hạn chế các nhược điểm của các hệ thống xử lý song song.
- Agile: Co thể hiểu là khả năng thay đổi nhanh chóng với các yêu cầu mới. Do cách thiết kế là Top Down, cho nên các thay đổi về mạt dữ liệu sẽ ít ảnh hưởng đến thiết kế của Data vault. Thông thường sẽ chỉ là thêm các bảng Satellites, Links. Nếu có phát sinh về business thì cũng chỉ là thêm các bảng Hubs và references.
- Structured, dễ dàng thay đổi (flexibility for refactoring)
- Có các khái niệm tương đồng với các mô hình khác -> dễ làm quen và nắm bắt cách dùng
- Hashed: Hash value là deterministic -> do đó có thể load các dữ liệu song song với nhau mà không cần tuân theo mối quan hệ của dữ liệu. Để rõ ràng hơn, khi ta sử dụng Surrogate Key là sequence, thì ta sẽ phải load dữ liệu theo các thứ tự Hubs -> Links -> Satellite. Phải load Hubs đầu tiên thì ta mới lấy được Key của Hubs sau đó các Bảng Links, Sagtellites sẽ sử dụng Key đó để refernce đến Hubs. Tuy nhiên nếu sử dụng Hash value, các giá trị này sẽ luôn là xác định (deterministic) -> Có thể load data parallel. Ngoài ra, nếu có sự cố hay phải chạy backfill thì cá key xây dựng từ Hash value sẽ được vẫn giữ nguyên giá trị. Việc sử dụng hash value còn làm tăng tốc độ xứ lý (Không cần xử lý concurrency), và các data warehouse hiện đại không còn hỗ trợ sequence nữa.
One huge advantage Data Vault has against other data warehousing architectures is that relationships can be added between Hubs with ease. Data Vault focuses on being agile and implementing what is needed to accomplish the current business goals. If relationships aren’t currently known or data sources aren’t yet accessible, this is ok because Links are easily created when they are needed. Adding a new Link in no way impacts existing Hubs or Satellites

Data Vault models are not built for consumption by business intelligence (BI) tools, they are built for **automation** and **agility** while allowing **auditability**; making changes to a Data Vault model does not destroy the existing model; rather, it augments it. In order to simplify and enhance querying a Data Vault model, we will discuss why you could consider building Point-in-Time (PIT) and Bridge tables.
## Cons
- Complex
- Quá nhiều join
- Cần hiểu rõ về business knowledge
# References
1. [How to implement data vault model - Aginic](https://aginic.com/blog/modelling-with-data-vaults/)
2. [Creating Data Vault Point-In-Time and Dimension tables: merging historical data sources - Roelant Vos](https://roelantvos.com/blog/creating-data-vault-point-in-time-and-dimension-tables-merging-historical-data-sources/)
3. [Data Vault Bridge tables (varigence.com)](https://docs.varigence.com/bimlflex/delivering-data-vault/data-vault-implementation-bridge)
4. [Data-Vault-Implementation-and-Automation-A-pattern-for-Data-Mart-delivery.pdf (roelantvos.com)](https://roelantvos.com/blog/wp-content/uploads/2019/01/Data-Vault-Implementation-and-Automation-A-pattern-for-Data-Mart-delivery.pdf)
5. [Point-in-Time (PIT) constructs & Join-Trees (snowflake.com)](https://www.snowflake.com/blog/point-in-time-constructs-and-join-trees/)
6. [Data Vault 2.0 and Big Data on waitingforcode.com - articles about General Big Data](https://www.waitingforcode.com/general-big-data/data-vault-2-big-data/read#architecture_examples)
7. [DataJoinery’s Data Vault Key Concepts – DataJoinery](https://datajoinery.io/datajoinerys-data-vault-key-concepts/)
8. [The Data Vault Guru: a pragmatic guide on building a data vault | LinkedIn](https://www.linkedin.com/pulse/data-vault-guru-pragmatic-guide-building-patrick-cuba/)