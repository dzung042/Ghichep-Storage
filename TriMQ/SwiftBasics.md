#Swift Basics

#Mục lục:
**Table of Content**

- [1. Giao tiếp với Cluster: Swift API](#1)
	- [1.1 Gửi yêu cầu](#11)
	- [1.2 Ủy quyền](#12)
	- [1.3 Phản hồi](#13)
	
- [2. Các tool giao tiếp](#2)

- [3. Ứng dụng client tùy chỉnh](#3)

- [4. Example Scenarios ](#4)

-----------------------------------------------------------
<a name="1"></a>
##1. Giao tiếp với Cluster: Swift API
Swift giành rất nhiều thời gian cho nhưng hoạt động, những yêu cầu gửi tới để đẩy dữ liệu lên các cluster, những dữ liệu này được sao chép và viết lên nhiều node khác nhau. Cũng có những yêu cầu lấy dữ liệu khỏi các cluster, như để khôi phục backup hoặc phụ vụ nội dung cho trang web chơi game trực tuyến. Đối với mỗi yêu cầu, Swift có thể kiểm tra xem ai được phép thực hiện yêu cầu trước khi nó được xử lý và đáp ứng. Ngoài ra thêm vào đó, các tiến trình server hay tiến trình consistency tràn ngập phía sau đòi hỏi phải có thông tin liên lạc và phối hợp giữa tất cả
Như đã đề cập ở chương 3, tiến trình máy chủ proxy là thành phần duy nhất giao tiếp với client bên ngoài. Bởi chỉ có mỗi tiến trình proxy thực thi được Swift API. Ở thời điểm này chúng ta có thể hiểu đơn giản rằng Swift API là tập HTTP dựa trên quy tắc, cú pháp từ vựng mà các tiến trình máy chỉ proxy dùng để giao tiếp với bên ngoài. Chúng ta sẽ tìm hiểu rõ hơn về API Swift ở chapter 5. Điểm nổi bật cảu tiến trình máy chủ proxy là tiến trình duy nhất có thể giao tiếp phía bên ngoài cluster, và nó sẽ lắng nghe cũng như thực thi HTTP
Sử dụng HTTP nghĩa giào sự giao tiếp được thực hiện theo dạng yêu cầu-phản hồi. Mỗi yêu cầu tương ứng với 1 mong muốn cụ thể (tải lên, ghi, sửa, ...) và được thể hiện với 1 động từ HTTP. Mỗi phản hồi trả về có 1 mã phản hồi như (200 OK) cho thấy những gì đã xảy ra đối với các yêu cầu, Vì quá trình giao tiếp với máy chủ prroxy HTTP, nên người dùng khi muốn giao tiếp chú ý tới
+ Việc gửi yêu cầu
+ Ủy quyền
+ Phản hồi

<a name="11"></a>
###1.1 Gửi yêu cầu
Nếu bạn gửi yêu cầu tới các Swift cluster, nó phải chứa đủ các thành phần sau:
- Storage URL
- Thông tin xác thực
- động từ HTTP
- Tùy chọn: dữ liệu hoặc metadata được ghi

#####Storage URL:
Các storage URL có 2 vai trò cần quan tâm: dó là cách yêu cầu gửi đến cluster và phải chỉ ra được vị trí trong cluster mà yêu cầu nhắm đến
Ví dụ, Storage URL có dạng
```sh
https://swift.example.com/v1/account/container/object
```
Nó sẽ gồm có 2 thành phần cơ bản:
- <b>Vị trí cluster (swift.example.com/v1)</b>: Phần đầu của Storage URL là 1 endpoint trong cluster. Nó được sửu dụng bởi việc mạng sẽ định tuyến yêu cyaf tới các node có tiến trình máy chủ proxy để yêu cầu của bạn có thể được sử lý
- <b>Vị trí lưu trữ (account/container/object)</b>: bao gồm 1 hoặc nhiều phần tạo nên sự duy nhất của vị trí dữ liệu. Các vị trí lưu trữ ccos thể là 1 trong 3 dạng tùy thuộc vào tài nnguyeen mà bạn đang có gắng gửi yêu cầu:
 + Account: /account
 + Container: /account/container
 + Object: /account/container/object
Lưu ý: Tên đối tượng có thể chứa ký tự gạch chéo (/) vì vậy việc giả lồng thư mục là hoàn toàn có thể xảy ra

#####Thông tin xác thực
Việc đầu tiên mà Swift làm là xác định xem yêu cầu đó đc ai gửi. Việc xác thực được thực hiện bằng cách so sánh các thông tin mà bạn cung cấp với thông tin máy chủ xác thức mà Swift sử dụng.
Có 2 cách để xác thực:
- Thông qua những thông tin xác thực ở mỗi lần gửi yêu cầu
- Thông qua xác thực token mà bạn có được bằng các yêu cầu xác thực trước khi yêu cầu về lưu trữ
Điều này tương tự như bạn đến thăm công ty cần có thông tin ID của bạn tại quầy kiểm tra mỗi khi bạn ra vào hoặc bạn có 1 thẻ riêng dùng để ra vào hàng ngày mà không cần lấy ID nhiều lần
Swift khi biết bạn là ai nó cũng sẽ ghi nhận việc những dữ liệu bạn được phép truy cập, Swift sử dụng thông tin này khi biết yêu cầu là cho đối tượng nào.
Có nhiều điều để nói về xác thực và ủy quyền, chi tiết sẽ ở trang 13. Đối với doanh nghiệp, điều quan trọng cần hiểu là mỗi yêu cầu lưu trữ phải có thông tin xác thực vì Swift sẽ xác minh xác thực mỗi

#####HTTP Verbs:
Chúng ta sẽ xem xét kỹ những hành động khác nhau khi yều cầu giao tiếp với HTTP. Swift sử dụng các động từ chuẩn HTTP như sau:
- GET: Download object(cùng metadata), hoặc list nội dung của container hoặc account
- PUT: upload object, tạo container hoặc ghi đè metadata headers.
- POST: cập nhật metadata (account hoặc container) ghi đè metadata(đối tượng) hoặc tạo container nếu nó chưa tồn tại
- DELETE: xóa object hoặc container trống
- HEAD: Lấy thông tin header, bao gồm metadata của account, container, object
Sau khi xác thực thành công, Swift sẽ xem xét các yêu cầu để xác định vị trí lưu tữ và hành động của yêu cầy, nó kiểm tra xem yêu cầu có được phép không.
Cụ thể hơn về ủy quyền

<a name="12"></a>
###1.2 Ủy quyền
Mặc dù người dùng có thể có thông tin hợp lệ để truy cập vào Swift cluster, nhưng họ có thể không được cấp phép cho những hành động mà họ gửi trong yêu cầu. Các máy chủ proxy sẽ cần xác nhận ủy quyền trước khi cho phép các yêu cầu để hoàn thành
Ví dụ: Nếu bản gửi 1 yêu cầu với thông tin hợp lệ, nhưng cố gắng thêm dữ liệu vào 1 account khác thì bạn vẫn sẽ được xác thực nhưng sẽ bị từ chối yêu cầu.
Nếu các hành động là được phép, các tiến trình máy chỉ proxy sẽ gọi đúng các node được yêu cầu, các node sẽ trả về kết quả của yêu cầu, và proxy sẽ trả về các phản hồi HTTP.

<a name="13"></a>
###1.3 Phản hồi
Các phản hồi từ Swift cluster sẽ chứa 2, hoặc thường là 3 các thành phần sau:
- Mã phản hồi và mô tả
- Header
- Data

Mã phản hồi và mô tả sẽ cho bạn biết biết yêu cầu của bạn được hoàn thành. Rộng ra có 5 mẫu mã phản hồi
```sh
- 1xx(thông tin): ví dụ 100 continue
- 2xx(thành công): ví dụ 200 OK hoặc 201 created
- 3xx(chuyển hướng): ví dụ 300 multiple choices hoặc 301 Moved Permanently
- 4xx(lỗi người dùng): ví dụ 400 bad request hoặc 401 là unauthorized
- 5xx(lỗi server): 500  Internal Server Error
```
Từ danh sách này ta có thể thấy mã 200 là dấu hiệu tốt thể hiện yêu cầu thành công. Các header đi kèm các phản hồi có thể cung cấp thêm thông tin, nếu yêu cầu là để GET dữ liệu, dữ liệu phải được trả về tốt.
Như bạn có thể thấy mặc dù định dạng yêu cầu-phản hồi là khá đơn giản, nhưng có rấy nhiều thành phần xung quanh đó để có thể đảm bảo ban giao tiếp được với các cluster.


<a name="4"></a>
##4. Các tool giao tiếp
Người dùng và admin thường quan tâm về yêu cầu-phản hồi HTTP sử dụng ứng dụng client. Ứng dụng Swift client có thể ở nhiều dạng, từ commandline giao diện đồ họa.
CLI là tất cả những gì bạn cần để thực hiển hoạt động đơn giản trên Swift cluster. BỞi vì ngôn ngữ tự nhiên của hệ thống Swift là HTTP, commande-lint tool như cURL là cách tốt nhất để giao tiếp vs Swift ở lớp thấp
Tuy nhiên, gửi nhiều yêu cầu HTTP cùng 1 lúc và giải nén các thông tin lên liên quan đến kết quả có thể có 1 chút vướng mắc. Vì lý do này mà mọi người thích sử dụng Swift ở tầng cao hơn. Như sử dụng thư viện client Swift thay vì làm việc với HTTP cơ bản

###Command-Line Interfaces
Chúng ta sẽ xem xét 2 câu lệnh cURL vs Swift. Cả 2 câu lệnh đều cho phép người dùng gửi yêu cầu 1 dòng 1 lần tới các Swift Cluster. 

####Sử dụng cURL (clinent for URL)
là câu lệnh phổ biến để chuyển dữ liệu đến và đi sử dụng các cấu trúc URL. Nó thường được cài đặt trước hoặc dẽ dàng được cài đặt bằng câu lệnh. cURL cung cấp chi tiết kiểm soát yêu cầy HTTP nên nó có thể xử lý tất cả các yêu cầu Swift. Vì cURL lấy các động từ HTTP 1 cách rõ ràng nên nó thường được sử dụng cung cấp ví dụ cho Swift
cURL gồm các thành phần sau:
```sh
+ curl
+ -X <method> tùy chọn chỉ ra động từ HTTP (get, post, put..)
+ thông tin xác thực
+ đường dẫn storage URL
+ Dữ liệu hoặc metadata (tùy chọn)

curl -X <HTTL-verb> [] <storage-url> <object.ext>
```
Chúng ta cùng xem xét 1 vài ví dụ về HTTP GET từ 1 người dùng tên Bob để xem cách cURL được sử dụng cho object, container, account. 1 cách phổ biến để sử dụng Swift là nơi mọi người dùng đều có chính xác 1 tài khoảnh. Chúng ra sẽ sử dụng mô hinhg đó ở đây, nên URL cho Bob sẽ là `http://swift.example.com/v1/AUTH_bob`. Đây sẽ là những tác vụ mà Bob sẽ thực hiên thông thường trong Swift.
```sh
- Tạo conainter mới: tạo lệnh put với vị trí container mới
 curl -X PUT [...] http://swift.example.com/v1/AUTH_bob/container2 
- List tất cả các container có trong account sử dụng get và chỉ ra địa chỉ account
 curl -X GET [...] http://swift.example.com/v1/AUTH_bob 
- Upload đối tượng: sử dụng PUT tới vị trí của đối tượng
 curl -X PUT [...] http://swift.example.com/v1/AUTH_bob/container1 -T object.jpg
- List các đối tượng trong container
 curl -X GET [...] http://swift.example.com/v1/AUTH_bob/container1 
- Download 1 đối tượng
 curl -X GET [...] http://swift.example.com/v1/AUTH_bob/container1/object.jpg 
```
####Sử dụng lệnh Swift
sử dụng Swift CLI là 1 phần của gói python-swiftclient và có thể được cài đặt hoặc trên những máy đã có python2.6 hoặc 2.7.
Giống như việc cURl sử dụng lệnh curl thì Swift cũng sử dụng lệnh swift. Lệnh Swift là đơn gian cho người dùng, bằng cách ngắn gọn hơn làm cho 1 số yêu cầu phổ biến trở nên đơn giản hơn. Tuy nhiên, sự đơn giản hóa này đi kèm với chi phí, lệnh Swift không thể làm tất cả những j mà hệ thống lưu trữ Swift có thể làm, Nó chỉ có 1 vài loại yêu cầu HTTP với câu lệnh Swift. Lý do khiến câu lệnh Swift phổ biến là vì nó cung cấp người sử dụng các động từ thông dụng với con người dùng để giao tiếp với cluster. Nó có thể dịch sang được ngôn ngữ HTTP. Điểm tối với câu lệnh này là nói yêu cầu bạn phải có thông tin xác thực với mỗi câu lệnh
Một số lệnh Swift tương ứng với HTTP như sau:
```sh
- delete(HTTP DELETE)
 xóa container hoặc object trong container
- download (HTTP GET)
 tải đối tượng trong container
- List (HTTP GET)
 list container trong account hoặc object trong container
- post (HTTP post)
cập nhật metadata cho account, container hoặc đối tượng (cũng có thể sử dụng để tạo container)
- stat(HTTP HEAD)
hiển thị thông tin header của account, container và đối tượng
- upload (HTTP PUT)
upload file hoặc thư mục lên container

curl -X GET [...] http://swift.example.com/v1/AUTH_bob/container1/object.jpg 

và 

swift download -U myusername -K mysecretpassword -A https://swift.example.com/auth/v1.0  http://swift.example.com/v1/AUTH_bob/container1/object.jpg
```
là tương tự nhau vì mỗi lần thực hiện yêu cầu Swift đều yêu cầu thông tin người dùng để xác thực nên ta tạm thời ký hiệu là [_] cho dễ đọc.
```sh
- List toàn bộ container trong account: 
swift list [...] http://swift.example.com/v1/AUTH_bob 
- List toàn bộ đối tượng trong container
swift list [...] http://swift.example.com/v1/AUTH_bob/container1 
- Tải về đối tượng
swift download [...] http://swift.example.com/v1/AUTH_bob/container1 object.jpg 
```
<a name="3"></a>
###3. Ứng dụng client tùy chỉnh
Mặc dù sử dụng lệnh là đủ cho các thao tác đơn gian, tuy nhiên nhiều người dùng muốn nhiều hơn các ứng dụng tinh vi vó thể tùy chỉnh và thích hợp. Các ứng dụng mà các developers có thể xây dựng yêu cầu HTTP và phân tích cú pháp http là các thư viện http của các ngôn ngữ trong đó có Python

<a name="4"></a>
###4. Example Scenarios 
Chúng ta sẽ xem xét qua 2 kịch bản là tải lên và tải về dữ liệu để có thể hiểu rõ hơn cách mà Swift hoạt động
- <b>Upload(PUT)</b>: 
Client sử dụng Swift APi để tạo yêu cầu HTTP PUT 1 đối tượng vào 1 container. Sau khi nhận yêu cầu PUT, tiến trình máy chủ proxy xác định vị trí dữ liệu hướng tới. Tên account, tên container, tên object, tất cả được sử dụng để xác định partion nơi mà đối tượng được lưu. 1 tra cứu trong rings thích hợp sẽ được sử dụng để map vị trí lưu trữ (/account/container/ object) tới partion và với tập các nút lưu trữ trong đó có mỗi bản sao của partion được phân công
Dữ liệu được gửi tới từng node lưu trữ, nơi mà có các partion thích hợp. Khi phần lớn dữ liệu được viết thành công, tiến trình máy chủ proxy có thể thông báo tới khách hàng quá trình upload đã thành công, VÍ dụ nếu bạn sử dụng 3 bản sao thì khi viết được 2 bản sao sẽ là thành công. Sau đó database container sẽ được cập nhật không đồng bộ để thể hiện các đối tượng mới trong container
- <b>Download (GET)</b>: 
1 yêu cầu đến tiến trình máy chủ proxy  /account/container/object Sửn dụng ring phù hợp để tra cứu, các partion của yêu cầu sẽ được xác định, cùng với tập hợp các node lưu trữ vùng đó. Mỗi yêu cầu sẽ được gửu đến các node lưu trữ để lấy tài nguyên. Sau khi yêu cầu trả về object thì tiến tình proxy sẽ trả về cho client.
