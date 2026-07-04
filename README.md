# SECURITY-LABS
Ứng dụng ELK Stack trong săn tìm mối đe dọa và ứng phó sự cố
ỨNG DỤNG ELK STACK TRONG SĂN TÌM MỐI ĐE DỌA VÀ ỨNG PHÓ SỰ CỐ

Khái niệm và Vai trò của IoC
Trong lĩnh vực an toàn thông tin và điều tra kỹ thuật số (computer forensics), Dấu hiệu xâm nhập (Indicator of Compromise - IoC) được định nghĩa là các bằng chứng kỹ thuật, dấu vết số hoặc dữ liệu quan sát được trên hệ thống máy chủ hoặc lưu lượng mạng mà với độ tin cậy cao, xác nhận rằng hệ thống đã bị xâm nhập hoặc bị tấn công bởi tác nhân đe dọa.   
Để chuẩn hóa khả năng phát hiện và đánh giá mức độ phức tạp của các IoC, mô hình "Pyramid of Pain" (Kim tự tháp Nỗi đau) do chuyên gia bảo mật David J. Bianco đề xuất năm 2013 được sử dụng rộng rãi như một khung tham chiếu chuẩn mực. Khung này phân loại các chỉ số phát hiện thành 6 tầng tăng dần theo mức độ khó khăn mà kẻ tấn công phải đối mặt nếu bị hệ thống phòng thủ chặn đứng:   
Hash Values (Giá trị băm): Các chữ ký số duy nhất của tệp tin (MD5, SHA-1, SHA-256). Đây là mức độ dễ thay đổi nhất đối với kẻ tấn công vì chỉ cần thay đổi một byte dữ liệu hoặc biên dịch lại mã nguồn là giá trị băm sẽ thay đổi hoàn toàn.   
IP Addresses (Địa chỉ IP): Các địa chỉ mạng của máy chủ điều khiển (C2), proxy hoặc máy chủ phát tán mã độc. Kẻ tấn công có thể dễ dàng thay đổi thông qua VPN, mạng Tor hoặc thay đổi hạ tầng máy chủ.   
Domain Names (Tên miền): Tên miền động được mã độc sử dụng để liên lạc. Việc thay đổi tên miền đòi hỏi thêm chi phí đăng ký và cập nhật DNS, gây ra một mức độ cản trở nhẹ.   
Network/Host Artifacts (Dấu vết mạng/hệ thống): Các dấu vết hoạt động đặc trưng để lại trên mạng (như chuỗi User-Agent lạ, cấu trúc URI đặc thù) hoặc trên máy trạm (như các registry key, tệp tin cấu hình tạm thời).   
Tools (Công cụ tấn công): Các phần mềm được kẻ tấn công sử dụng (như Mimikatz, Cobalt Strike, Metasploit). Khi một công cụ bị phát hiện và chặn đứng, kẻ tấn công buộc phải tìm kiếm, phát triển hoặc học cách sử dụng một công cụ thay thế, gây gián đoạn đáng kể đến chiến dịch. 
Tactics, Techniques, and Procedures - TTPs (Chiến thuật, Kỹ thuật và Quy trình): Cách thức cốt lõi mà kẻ tấn công vận hành. Việc buộc kẻ tấn công phải thay đổi TTPs là khó khăn nhất vì nó đánh thẳng vào thói quen, kỹ năng và quy trình hoạt động của tổ chức tội phạm mạng.   
Ví dụ về IoC
Các dấu hiệu xâm nhập IoC thường được phân chia thành ba nhóm chính dựa trên nguồn telemetry thu thập được:   
IoC dựa trên mạng (Network-based IoCs): Bao gồm các yêu cầu DNS bất thường đến các tên miền lạ (Anomalous DNS Requests), lưu lượng dữ liệu kết nối hướng ngoại tăng đột biến một cách bất thường (Unusual Outbound Network Traffic) biểu thị cho hành vi rò rỉ dữ liệu, hoặc các kết nối TCP/UDP không xác định tới các dải IP đen.   
IoC dựa trên máy trạm (Host-based/File-based IoCs): Bao gồm sự xuất hiện của các tệp tin thực thi lạ nằm tại các thư mục hệ thống nhạy cảm (như C:\Windows\System32\ hoặc AppData), các tiến trình hệ thống bị chèn mã độc (Process Injection), các chỉnh sửa trái phép đối với khóa Registry khởi động nhằm duy trì sự hiện diện, hoặc sự vô hiệu hóa đột ngột của dịch vụ Windows Defender/Antivirus.   
IoC dựa trên hành vi (Behavioral IoCs): Gồm các chuỗi sự kiện đăng nhập thất bại liên tiếp trong thời gian ngắn (Brute Force), việc thực thi các lệnh hệ thống đáng ngờ từ các tiến trình không có thẩm quyền (như cmd.exe hoặc powershell.exe được khởi tạo từ một trình duyệt web hoặc phần mềm gõ tiếng Việt UnikeyNT).   
Thiết lập và chuẩn bị môi trường giám sát
Để thực nghiệm quy trình thu thập, phân tích và phát hiện các hành vi xâm nhập thông qua giải pháp ELK Stack kết hợp telemetry từ Sysmon, một mô hình Lab được xây dựng với các thành phần cấu trúc như sau:

Cấu hình giám sát sự kiện trên Windows 11 bằng Sysmon
Hệ điều hành Windows 11 được sử dụng làm máy trạm Client cho người dùng cuối. Để nâng cao độ chi tiết của nhật ký hệ thống vượt trội so với Windows Event Log mặc định, công cụ Sysmon (System Monitor) thuộc bộ tiện ích Microsoft Sysinternals được cài đặt. Sysmon hoạt động dưới dạng một dịch vụ hệ thống và trình điều khiển thiết bị (device driver) để ghi nhận liên tục các hành vi cốt lõi như khởi tạo tiến trình, kết nối mạng, thay đổi tệp tin.
Cài đặt và cấu hình Winlogbeat để chuyển tiếp nhật ký
Winlogbeat là một tác nhân (shipper) hạng nhẹ được phát triển bởi Elastic, chuyên dùng để thu thập và chuyển tiếp nhật ký sự kiện Windows một cách tối ưu. Trên máy Windows 11, Winlogbeat được cấu hình để đọc dữ liệu từ các kênh log truyền thống kết hợp với kênh dữ liệu nâng cao của Sysmon, sau đó đẩy về bộ phân tích cú pháp Logstash chạy trên máy chủ Ubuntu.   
Tệp tin cấu hình chính winlogbeat.yml được chỉnh sửa loại bỏ đầu ra trực tiếp tới Elasticsearch và thiết lập đầu ra hướng tới dịch vụ Logstash (lắng nghe tại cổng 5044):   
Thiết lập máy Client chứa ứng dụng UnikeyNT bị chèn Trojan
Trên máy trạm Windows 11 (192.168.10.129), một phiên bản phần mềm gõ tiếng Việt UnikeyNT (tên tệp tin: UnikeyNT.exe) đã bị kẻ tấn công can thiệp, đóng gói kèm theo phần mềm bị chèn Trojan (Trojanized Software). Về mặt giao diện và tính năng gõ tiếng Việt, tệp tin này vẫn hoạt động hoàn toàn bình thường để đánh lừa nhận thức của người dùng, tuy nhiên phần mã độc chạy ngầm bên dưới sẽ tự động kích hoạt ngay khi tiến trình của ứng dụng được khởi chạy.

Cấu hình trình lắng nghe trên máy Kali Linux bằng Metasploit
Hệ điều hành Kali Linux đóng vai trò là máy tấn công của hacker. Công cụ Metasploit Framework được khởi chạy để thiết lập một trình lắng nghe kết nối ngược (Reverse TCP Listener). Mô-đun phản hồi sự cố được cấu hình cụ thể như sau:

(Trong đó 192.168.10.130 là địa chỉ IP của máy Kali Linux đóng vai trò Command & Control - C2 Server, lắng nghe kết nối ngược trên cổng 1234).
Cấu hình máy chủ phân tích Ubuntu ELK Stack
Hệ điều hành Ubuntu Server (192.168.10.132) được triển khai cài đặt bộ giải pháp phần mềm ELK Stack (bản phân phối chạy bằng Docker-compose để đảm bảo tính đồng bộ phiên bản).   
Logstash: Được cấu hình một đường ống (pipeline) tại tệp tin logstash.conf để chấp nhận dữ liệu đầu vào định dạng Beats thông qua cổng 5044, đồng thời định tuyến đầu ra đẩy trực tiếp vào Elasticsearch để lập chỉ mục (indexing).   
Elasticsearch: Lưu trữ cơ sở dữ liệu nhật ký tập trung và hỗ trợ tìm kiếm phân tích thời gian thực.   
Kibana: Giao diện trực quan hóa kết nối với Elasticsearch qua cổng 5601 phục vụ SOC Analyst truy vết.   
Quy trình tấn công và thu thập Indicators

Tiến trình mô phỏng cuộc tấn công thực tế và cơ chế ghi nhận nhật ký của hệ thống giám sát được mô tả chi tiết qua ba phân đoạn:
Hành vi kích hoạt mã độc từ phía người dùng Client
Do sự chủ quan, thiếu nhận thức về an toàn thông tin và không thực hiện kiểm tra tính toàn vẹn của tệp tin trước khi sử dụng (ví dụ: đối chiếu mã băm SHA-256), người dùng trên máy trạm Windows 11 (192.168.10.129) đã tải xuống và nhấp đúp để mở tệp tin mã độc UnikeyNT.exe giả mạo. Tiến trình này được khởi chạy trực tiếp với quyền hạn của người dùng hiện hành (User Privilege).
Tiến trình khai thác và kiểm soát từ phía Kali Linux
Ngay khi người dùng kích hoạt tệp tin, quy trình tấn công ba bước của hacker được kích hoạt tự động:

Khai thác (Exploitation): Giai đoạn này không tấn công lỗ hổng hệ thống mà khai thác yếu tố con người qua kỹ thuật Social Engineering. Khi người dùng khởi chạy tệp nhị phân giả mạo (Trojanized Binary) UnikeyNT.exe dưới quyền của mình, mã độc sẽ tự giải nén (unpack) và nạp trực tiếp payload Meterpreter vào không gian bộ nhớ hệ thống. Cơ chế thực thi trên bộ nhớ (In-memory execution) này giúp payload né tránh các giải pháp phát hiện mã độc truyền thống để chuẩn bị thiết lập kênh điều khiển ngược. 
Kết nối (Connection): Payload kích hoạt một tiến trình ngầm gửi yêu cầu kết nối mạng hướng ngoại TCP (Reverse Connection) từ máy trạm Client (192.168.10.129) hướng đến máy chủ của hacker tại IP 192.168.10.130 qua cổng 1234. Yêu cầu này nhanh chóng vượt qua tường lửa nội bộ (Firewall) của Client vì tường lửa thường cho phép các kết nối hướng ra (outbound) một cách mặc định. Phiên làm việc (Meterpreter Session) được thiết lập thành công trên máy Kali Linux.
Thao tác sau khai thác (Actions on Object): Kẻ tấn công có toàn quyền tương tác với máy trạm của nạn nhân thông qua bảng điều khiển dòng lệnh của Meterpreter (như tải/xuất tệp tin trái phép, chụp màn hình, ghi nhận thao tác bàn phím - keylogger, hoặc thực hiện kỹ thuật di chuyển ngang - lateral movement trong hệ thống mạng nội bộ).
Thu thập và chuyển tiếp log của Winlogbeat
Song song với các hành vi mạng của mã độc trên máy trạm, hệ thống phòng thủ thụ động hoạt động liên tục:
Dịch vụ Sysmon ghi nhận hành vi kết nối mạng TCP từ một tiến trình không thuộc nhóm ứng dụng trình duyệt hoặc dịch vụ hệ thống chuẩn. Hành vi này ngay lập tức kích hoạt sự kiện Sysmon Event ID 3 (Network Connection detected).   
Dịch vụ Winlogbeat chạy nền định kỳ quét tệp tin nhật ký sự kiện của Windows, phát hiện bản ghi mới xuất hiện trong kênh log của Sysmon.   
Winlogbeat lập tức đóng gói toàn bộ cấu trúc dữ liệu của sự kiện này thành định dạng JSON tiêu chuẩn hóa ECS (Elastic Common Schema) và truyền tải qua cổng mạng 5044 đến dịch vụ Logstash trên máy chủ Ubuntu ELK.   
Phân tích và phát hiện dấu vết tấn công trên Kibana
Sau khi dữ liệu log được Logstash tiếp nhận, chuẩn hóa cấu trúc và lưu trữ tập trung tại Elasticsearch, chuyên viên phân tích bảo mật (SOC Analyst) tiến hành khai thác giao diện Kibana để săn tìm mối đe dọa (Threat Hunting).
Truy vấn lưu lượng mạng qua Sysmon Event ID 3
Trong cơ chế ghi nhận của Sysmon, Event ID 3 (Network connection detected) là sự kiện chuyên biệt dùng để lưu giữ thông tin chi tiết về các kết nối TCP/UDP được khởi tạo bởi các tiến trình nội bộ trên hệ điều hành. Do mã độc Meterpreter bắt buộc phải duy trì một kết nối mạng liên tục về máy chủ điều khiển C2 của kẻ tấn công để nhận lệnh, sự kiện Event ID 3 là chìa khóa then chốt để phát hiện hành vi này.   
Tại giao diện Kibana Discover, chuyên viên phân tích thực hiện bộ lọc tìm kiếm theo mã định danh sự kiện của Sysmon:

Để trực quan hóa bức tranh toàn cảnh về lưu lượng mạng hướng ngoại, một biểu đồ hình tròn (Pie Chart) phân tích phân phối các địa chỉ IP đích (Destination IP) được xây dựng trên Kibana Dashboard.


Nhận diện và điều tra địa chỉ IP bất thường
Thông qua biểu đồ hình tròn, chuyên viên phân tích nhanh chóng nhận thấy một lượng lớn các giao dịch kết nối mạng hướng ngoại liên tục đổ về một địa chỉ IP lạ 192.168.10.130. Địa chỉ này không thuộc các IP nội bộ hợp lệ của doanh nghiệp, không thuộc các dải dịch vụ Public Cloud tin cậy (như AWS, Azure, Google Cloud) và không có bản ghi định danh tên miền hợp pháp.
Để làm rõ bản chất kỹ thuật của hành vi đáng ngờ này, chuyên viên phân tích tiến hành lọc sâu sự kiện mạng liên quan đến địa chỉ IP trên bằng truy vấn:

Dựa trên dữ liệu cấu trúc trả về từ Elasticsearch, hai trường thông tin kỹ thuật cốt lõi sau được phân tích chi tiết:
1. Trường dữ liệu winlog.event_data.DestinationPort
Định nghĩa kỹ thuật: Đây là trường lưu giữ thông tin về cổng đích (Destination Port) của giao dịch mạng được khởi tạo từ máy Client.   
Vai trò trong điều tra: Giúp xác định dịch vụ hoặc giao thức truyền thông mà tiến trình đang cố gắng thiết lập kết nối. Trong kịch bản thực tế, trường này trả về giá trị là 1234. Cổng 1234 là cổng dịch vụ không chuẩn (non-standard port), thường không được gán cho bất kỳ dịch vụ CNTT chính thống nào. Việc một máy trạm nội bộ liên tục kết nối đến một IP lạ qua cổng 1234 là một Dấu hiệu xâm nhập mạng (Network IoC) cực kỳ nghiêm trọng.   

2. Trường dữ liệu winlog.event_data.Image
Định nghĩa kỹ thuật: Trường dữ liệu này lưu giữ đường dẫn tuyệt đối của tệp tin thực thi (executable path) của tiến trình chịu trách nhiệm khởi tạo hoặc tương tác với sự kiện mạng tương ứng.   
Vai trò trong điều tra: Cung cấp mối liên kết trực tiếp giữa hành vi mạng bất thường với một thực thể phần mềm cụ thể trên đĩa cứng. Trong kết quả truy vấn, trường này trả về giá trị: C:\Users\User\Desktop\UnikeyNT.exe.

Đánh giá và Kết luận điều tra:
Đối với một môi trường vận hành tiêu chuẩn, bộ gõ tiếng Việt UnikeyNT là một ứng dụng tiện ích văn phòng thuần túy, hoạt động hoàn toàn ngoại tuyến (offline) và không có bất kỳ lý do kỹ thuật hợp lệ nào để khởi tạo một kết nối mạng ra bên ngoài hệ thống.
Việc trường winlog.event_data.Image chỉ ra tiến trình UnikeyNT.exe đang liên kết với trường winlog.event_data.DestinationPort có giá trị 1234 hướng tới IP máy lạ 192.168.10.130 là bằng chứng (Dấu hiệu xâm nhập hành vi - Behavioral IoC) khẳng định ứng dụng UnikeyNT trên máy trạm đã bị chèn mã độc Trojan kết nối ngược (Reverse Shell). Kẻ tấn công đã hoàn thành việc chiếm quyền kiểm soát máy trạm của người dùng.   

Dựa trên bằng chứng thu thập được từ ELK Stack, quản trị viên an ninh mạng có thể lập tức đưa ra phương án xử lý sự cố khẩn cấp:
Cô lập (Containment): Cách ly máy trạm bị lây nhiễm (192.168.10.129) khỏi mạng nội bộ để ngăn chặn di chuyển ngang.
Loại bỏ (Eradication): Xóa tệp tin thực thi độc hại để phân tích mã nguồn (reverse engineering), đồng thời cập nhật quy tắc tường lửa chặn toàn bộ lưu lượng đến địa chỉ IP 192.168.10.130 
Hồi phục hệ thống (Recovery): Hệ thống đã được khôi phục sau đó sẽ được tích hợp trở lại vào quy trình kinh doanh mà nó hỗ trợ
Rút kinh nghiệm (Lesson learned): Phân tích sự cố và các phản ứng để xác định xem các quy trình hoặc hệ thống có thể được cải thiện không. Cũng rất quan trọng để ghi lại tài liệu về sự cố.

