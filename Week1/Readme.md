# **Week 1 Report**

**Intern:** Nguyễn Như Thuật

**Team:** Platform \- Adtech

**Duration:** Week 1

# **1\. Overview**

Trong tuần đầu tiên, em tập trung vào hai nhóm kiến thức nền tảng cho Java Backend: **Java Core** và **Linux Fundamentals**. Với Java Core, em ôn lại các khái niệm quan trọng như OOP, SOLID, Interface, Abstract Class, Static, Collections, thread-safe alternatives và Exception Handling. Với Linux, em tìm hiểu các nhóm lệnh cơ bản phục vụ công việc backend như quản lý file, phân quyền, process, xử lý text, network utilities, system monitoring, shell scripting và system logs.

Qua tuần này, em nhận ra Java Core giúp em xây dựng tư duy thiết kế code, còn Linux giúp em hiểu môi trường mà ứng dụng backend sẽ chạy thực tế. Vì vậy, em không chỉ học cú pháp hoặc học lệnh riêng lẻ, mà cố gắng hiểu vấn đề phía sau bằng cách **đưa vào thực chiến**: vì sao cần SOLID, khi nào dùng Interface, vì sao HashMap không thread-safe (gây bán lố vé), hoặc vì sao backend developer cần biết dùng Pipeline phân tích log và theo dõi tài nguyên hệ thống.

# **2\. Learning Summary**

| Chủ đề                        | Các đầu mục                                                  | Kiến thức đã thu thập                                                                                                                   | Trạng thái                                                                          |
|:------------------------------|:-------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------------|
| OOP & SOLID                   | Encapsulation, Abstraction, Inheritance, Polymorphism, SOLID | OOP là nền tảng để tổ chức object, còn SOLID là nguyên tắc kiến trúc. Tránh God Class, mở rộng tính năng không cần sửa code cũ.         | Nắm vững & Đã code thực tế                                                          |
| Interface / Abstract / Static | CAN-DO vs IS-A, static thuộc về class                        | Hiểu khi nào nên dùng Interface (ép buộc hành vi). Nhận thức được rủi ro khi dùng Static cho dữ liệu động ở môi trường Multi-thread.    | Nắm vững                                                                            |
| Collections                   | ArrayList, LinkedList, HashMap, Thread-safe alternatives     | Hiểu Internals của HashMap (Hashing, Collision). Hiểu rõ vì sao phải dùng ConcurrentHashMap cho môi trường đa luồng (High concurrency). | Nắm vững & Đã thực hành                                                             |
| Exception Handling            | Checked/Unchecked, try-catch, Custom Exception               | Bắt lỗi cụ thể, không nuốt lỗi, tự tạo Custom Exception để tách biệt lỗi logic nghiệp vụ và lỗi hệ thống.                               | Đã thực hành                                                                        |
| Linux Permissions             | File system, chmod, chown, umask                             | Hiểu bản chất cộng điểm phân quyền (Read=4, Write=2, Execute=1) để áp dụng chính xác thay vì gõ vẹt chmod 777\.                         | Đã thực hành                                                                        |
| Text Processing & I/O         | grep, awk, sort, uniq, Pipeline                              | Nắm được sức mạnh của Pipeline (\`                                                                                                      | \`), biến output lệnh này thành input lệnh kia để phân tích Log hệ thống cực nhanh. |
| Monitoring & Scripting        | Process, monitoring, Shell Script                            | Biết cách tìm/kill process treo, xử lý đầy ổ cứng, viết script .sh để tự động hóa việc giám sát tài nguyên.                             | Đã thực hành                                                                        |

# **3\. Learning Details**

## **3.1 Java Core**

Trong tuần này, em ôn lại các phần nền tảng của Java Core. Thay vì chỉ ghi nhớ định nghĩa, em đã áp dụng vào các bài toán cụ thể để hiểu cách thiết kế và maintain code.

* Với **OOP**, em hiểu rõ hơn vai trò của 4 tính chất chính thông qua việc tổ chức dữ liệu.
    * *Ví dụ thực chiến:* Để bảo vệ tính toàn vẹn của dữ liệu, em áp dụng Encapsulation khi thiết kế hệ thống tài khoản. Thay vì để public, em đóng gói biến số dư và kiểm soát logic rút tiền:

public class BankAccount {  
private double balance; // Đóng gói dữ liệu

     // Không cung cấp hàm setBalance() để tránh bị can thiệp sai lệch
    public void withdraw(double amount) {  
        if (amount <= 0 || amount > balance) {  
            throw new InsufficientFundsException("Số tiền không hợp lệ hoặc không đủ số dư\!");  
        }  
        this.balance -= amount;  
    }  
}


* Với **SOLID**, em hiểu đây là các nguyên tắc giúp code dễ bảo trì khi yêu cầu nghiệp vụ thay đổi.
    * *Ví dụ thực chiến:* Khi xây dựng hệ thống **Lọc Sản Phẩm**, thay vì dùng if-else vi phạm Open/Closed Principle, em định nghĩa một Interface (hợp đồng lọc). Bất kỳ tiêu chí mới nào cũng chỉ cần tạo class mới:

public interface ProductSpecification {  
boolean isSatisfied(Product p);  
}

// Mở rộng tính năng bằng cách thêm class, không sửa code cũ  
public class PriceSpecification implements ProductSpecification {  
private double threshold;  
public PriceSpecification(double threshold) { this.threshold = threshold; }  
public boolean isSatisfied(Product p) { return p.getPrice() > threshold; }  
}

* Với **Interface, Abstract Class và Static**, em phân biệt theo bản chất: Abstract Class phù hợp với quan hệ **IS-A** (kế thừa thuộc tính), còn Interface phù hợp với khả năng **CAN-DO** (ép buộc thực thi một hành vi). Em lưu ý tránh lạm dụng biến static để lưu dữ liệu thay đổi liên tục vì dễ gây lỗi data racing.
* Với **Collections & Thread-safe**, em đào sâu vào sự đánh đổi hiệu năng và bài toán đồng bộ hóa.
    * *Ví dụ thực chiến:* Em mô phỏng hệ thống **Flash Sale** (10 vé, 100 luồng mua). Việc dùng ArrayList gây "bán lố vé". Em đã tái cấu trúc, sử dụng AtomicInteger và ConcurrentHashMap (tận dụng khóa phân vùng \- Node lock) để đảm bảo an toàn:

public class FlashSale {  
// Thread-safe: Đảm bảo số lượng vé trừ đi chính xác  
private AtomicInteger tickets = new AtomicInteger(10);

    // Thread-safe: Đảm bảo luồng đọc/ghi danh sách người mua không đụng độ  
    private Map<String, String> successfulBuyers = new ConcurrentHashMap<>();

    public void buyTicket(String userId) {  
        if (tickets.getAndDecrement() > 0\) {  
            successfulBuyers.put(userId, "SUCCESS");  
        }  
    }  
}

* Với **Exception Handling**, em hiểu giá trị của việc bắt lỗi có chủ đích. Trong đoạn code withdraw() ở trên, thay vì ném ra lỗi chung chung, em định nghĩa InsufficientFundsException. Việc này giúp Controller bắt đúng lỗi và trả về thông báo rõ ràng cho frontend.

## **3.2 Linux Fundamentals**

Ở phần Linux, em tập trung vào các nhóm lệnh vận hành và debug ứng dụng backend.

* Với **File System và Permissions**, em hiểu cách hệ điều hành bảo vệ file dựa trên công thức tính điểm.
    * *Thực hành:* Thiết lập vùng an toàn tuyệt đối cho thư mục khóa bảo mật và kiểm tra lại kết quả:


mkdir secret_keys  
chmod 700 secret_keys # Chỉ User sở hữu mới có quyền đọc/ghi/thực thi (rwx------)

# Kiểm tra lại quyền của thư mục
ls -ld secret_keys

Output thực tế:  
drwx------ 2 ubuntu ubuntu 4096 Oct 10 10:00 secret_keys

* Với **Process Management**, em học cách cô lập và xử lý các tiến trình cứng đầu.
    * *Thực hành:* Giả lập sự cố "Port 8080 already in use" khi chạy Spring Boot:

# Tìm PID đang chiếm port 8080
ss -tulpn | grep 8080   
Output: tcp   LISTEN 0 100 *:8080 *:* users:(("java",pid=1234,fd=18))

# Ép đóng tiến trình (ví dụ PID là 1234\)
kill -9 1234

# Chạy ngầm ứng dụng an toàn
nohup java -jar app.jar &  
Output: [1] 1235 (Kèm file nohup.out lưu log)

* Với **Text Processing và I/O Redirection**, em kết hợp các lệnh để trích xuất dữ liệu từ file log.
    * *Thực hành:* Phân tích file access.log, tìm IP tấn công spam nhiều nhất bằng Pipeline:

# Lấy IP (cột 1) -> Sắp xếp -> Đếm gom nhóm -> Xếp hạng từ cao xuống thấp
awk '{print $1}' access.log | sort | uniq -c | sort -nr

Output thực tế:  
150 192.168.1.5    
20 10.0.0.2

* Với **Network, Monitoring và Logs**, em tìm hiểu cách "bắt mạch" server.
    * *Thực hành:* Khi server báo lỗi đầy ổ cứng (Disk 100%), em quét hệ thống bằng trick lọc lỗi an toàn (chuyển hướng stderr 2 vào thùng rác):

# Tìm thư mục chiếm dung lượng lớn nhất mà không bị rác màn hình
du -sh * 2>dev/null | sort -h

Output thực tế:  
...  
500M    /home  
4.5G    /var/log  <-- Tìm ra thủ phạm

# **4\. Practice**

Trong tuần này, thay vì chỉ đọc hiểu, em đã tự tay build và chạy thử các bài tập thực chiến từ Level 1 đến Level 3 để củng cố toàn bộ kiến thức:

**Phần Java Core:**

* **[Level 1] Khởi động với OOP & Exception:** Code hệ thống BankAccount, vận dụng Encapsulation để bảo vệ luồng dữ liệu. Tự định nghĩa InsufficientFundsException và áp dụng vào Controller giả lập để quản lý ngoại lệ thay vì crash ứng dụng.  
  public static void main(String[] args) {  
  BankAccount acc = new BankAccount(50000);   
  try {  
  acc.withdraw(100000); // Thử rút vượt số dư  
  } catch (InsufficientFundsException e) {  
  System.out.println("❌ Cảnh báo nghiệp vụ: " + e.getMessage());  
  }  
  }

* **[Level 2] Design Pattern & OCP:** Tái cấu trúc hệ thống **Product Filter** từ cấu trúc if-else tĩnh sang Interface Specification. Việc này cho phép truyền động các bộ lọc vào runtime mà không cần chỉnh sửa logic lõi.  
  List<Product> products = Arrays.asList(new Product("Laptop", 1500), new Product("Phone", 800));

  // Dễ dàng thay thế điều kiện lọc (Giá, Danh mục, Màu sắc) qua tính Đa hình  
  List<Product> expensiveItems = filter(products, new PriceSpecification(1000));

* **[Level 3] Multi-threading & Thread-safe:** Mô phỏng kịch bản **Flash Sale**. Em đã thử nghiệm việc giả lập hàng trăm luồng request đồng thời để quan sát lỗi Race Condition, sau đó fix hoàn toàn bằng AtomicInteger và ConcurrentHashMap.  
  FlashSale sale = new FlashSale();

  // Giả lập 100 threads đồng thời mua (trong khi chỉ có 10 vé)  
  for (int i = 0; i < 100; i++) {  
  final int userId = i;  
  new Thread(() -> sale.buyTicket("User_" + userId)).start();  
  }  
  // Kết quả sau tối ưu: Số lượng người mua thành công luôn chính xác là 10\.

**Phần Linux & Scripting:**

* **[Level 1] Phân quyền & Process Management:** Khởi tạo thư mục mật, thiết lập quyền an toàn và giả lập kịch bản treo terminal để thực hành chạy nền tiến trình ngầm.  
  mkdir secret_data && chmod 700 secret_data  
  ls -ld secret_data  
  Output: drwx------ 2 user group 4096 Oct 10 10:00 secret_data

  sleep 3600 &  
  Output: [1] 5678 (Chạy ngầm với PID 5678)

  # Kiểm tra PID và tiêu diệt tiến trình
  kill -9 $(ps aux | grep '[s]leep' | awk '{print $2}')

* **[Level 2] Text Processing Pipeline:** Dùng Pipeline kết hợp các tool mạnh mẽ để parse log thô sơ.
# Trích xuất và thống kê xem ngày nào hệ thống bắn ra nhiều lỗi ERROR nhất
grep "ERROR" server.log | awk '{print $2}' | sort | uniq -c | sort -nr

Output mong đợi:
15 2023-10-02  
3 2023-10-01

* **[Level 3] Tự động hóa SysAdmin (Shell Script):** 1. Viết script check\_health.sh: Kết hợp free -m để cảnh báo hệ thống khi RAM tụt dưới ngưỡng cho phép.
    2. Viết script clean_logs.sh: Tìm các file log quá hạn (> 7 ngày), nén lại để backup và xóa file gốc định kỳ.  
       \#\!/bin/bash  
       LOG_DIR="/var/log/myapp"

  # Nén các file log cũ hơn 7 ngày để giải phóng dung lượng đĩa
  find $LOG_DIR -name "*.log" -mtime +7 -exec tar -czvf {}.tar.gz {} \\;

  # Dọn dẹp file gốc
  find $LOG_DIR -name "*.log" -mtime +7 -exec rm {} \\;  
  echo "✅ Hoàn tất dọn dẹp log định kỳ!"

# **5\. Reflection and Difficulties**

Trong tuần đầu tiên, em nhận ra khó khăn lớn nhất là lượng kiến thức khá rộng. Java Core tập trung vào tư duy thiết kế code, còn Linux lại tập trung vào môi trường vận hành và các lệnh thực tế. Nếu chỉ học theo kiểu ghi nhớ định nghĩa hoặc nhớ lệnh, em rất dễ quên.

Vì vậy, em cố gắng học bằng cách tự đặt câu hỏi: *"Kiến thức này giải quyết vấn đề gì?"*, *"Nếu dùng sai thì hậu quả là gì?"*, *"Trong backend thực tế sẽ gặp ở đâu?"*.

Ví dụ, với SOLID, em không chỉ nhớ tên từng chữ cái mà thông qua bài tập Lọc Sản Phẩm, em thực sự hiểu việc không dùng Interface sẽ khiến code chằng chịt if-else khó bảo trì. Với môi trường Multi-thread, em thấy rõ hậu quả sai lệch data nếu dùng sai Collection. Với Linux, em không chỉ nhớ lệnh awk hay grep, mà thông qua kịch bản phân tích log và đầy ổ cứng, em thấy chúng là công cụ cứu cánh khi server gặp sự cố.

Một số phần em vẫn cần luyện thêm trong các tuần tiếp theo là áp dụng Design Patterns vào hệ thống phức tạp hơn, viết shell script nâng cao và sử dụng Mockito/JUnit để viết Unit Test bảo chứng chất lượng code.