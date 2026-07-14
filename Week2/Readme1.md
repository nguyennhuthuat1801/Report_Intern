# **Week 2 Report**

**Intern:** Nguyễn Như Thuật

**Team:** Platform \- Adtech

**Duration:** Week 2

# **1\. Overview**

Tiếp nối tuần đầu tiên, trong Tuần 2, em tập trung vào việc nâng cấp tư duy thiết kế phần mềm và đảm bảo chất lượng code với 3 chủ đề chính: **Collections Internals**, **Design Patterns**, và **Unit Testing (JUnit & Mockito)**.

Thay vì chỉ biết cách "sử dụng" các class có sẵn, em đã đi sâu vào tìm hiểu **cơ chế hoạt động bên dưới (Internals)** của chúng (ví dụ: HashMap lưu dữ liệu ra sao, khi nào thì dùng mảng, khi nào dùng cây Red-Black). Đồng thời, em áp dụng các **Design Patterns** kinh điển để giải quyết các bài toán lặp lại một cách thanh lịch, tránh việc code bị "phình to" (God Class) hay dính chặt vào nhau (Tight Coupling). Cuối cùng, để bảo chứng cho các đoạn logic nghiệp vụ, em đã thực hành viết Unit Test độc lập bằng cách sử dụng kỹ thuật **Mocking**.

Những kiến thức tuần này đặc biệt quan trọng khi làm việc tại team Platform \- Adtech, nơi hệ thống đòi hỏi tính linh hoạt cao để thêm mới các format quảng cáo (áp dụng Factory) và cần độ tin cậy tuyệt đối qua các bộ Test Coverage.

# **2\. Learning Summary**

| Chủ đề                       | Đầu mục                                            | Kiến thức đã thu thập                                                                                                                                | Trạng thái            |
|:-----------------------------|:---------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------|
| Collections Internals        | ArrayList, HashMap, Hash Collision, Red-Black Tree | Hiểu bản chất cấu trúc dữ liệu bên dưới, cơ chế tự phình to của mảng và cách Java 8 tối ưu HashMap bằng Red-Black Tree.                              | Nắm vững              |
| Java Contract                | equals() và hashCode()                             | Nắm được "luật bất thành văn" BẮT BUỘC phải Override cả 2 hàm này khi dùng Custom Object làm Key trong Map/Set để tránh trùng lặp dữ liệu bộ nhớ.    | Nắm vững & Đã fix bug |
| Design Patterns (Creational) | Singleton, Factory Method                          | **Singleton:** Đảm bảo duy nhất 1 instance (vận dụng Double-checked locking). **Factory:** Khởi tạo object động không cần dùng từ khóa new tràn lan. | Nắm vững & Đã code    |
| Design Patterns (Behavioral) | Observer Pattern                                   | Hiểu cơ chế Publish-Subscribe, giúp các module giao tiếp lỏng lẻo (loosely coupled), dễ dàng trigger nhiều action khi 1 event xảy ra.                | Đã thực hành          |
| Unit Testing                 | FIRST Principles, TDD                              | Nắm được nguyên tắc viết test: Fast, Isolated, Repeatable, Self-validating, Timely. Test không phụ thuộc vào thứ tự chạy.                            | Nắm vững              |
| Mocking                      | Mockito, @Mock, @InjectMocks                       | Hiểu tại sao phải Mock (cô lập logic, không chọc vào DB thật). Biết cách "dạy" Mock object trả về kết quả giả lập để test luồng nghiệp vụ.           | Đã thực hành          |

# **3\. Learning Details**

## **3.1 Collections Internals & The Java Contract**



* **ArrayList:** Bản chất là một mảng tĩnh (Array). Khi đầy, nó tự tạo mảng mới lớn gấp 1.5 lần và copy data sang. Bài học rút ra: Nếu biết trước số lượng phần tử (ví dụ import 10,000 dòng file Excel), phải khởi tạo dung lượng ngay từ đầu new ArrayList\<\>(10000) để tránh tốn CPU cho việc resize liên tục.
* **HashMap & Hash Collision:** HashMap dùng mảng các "Bucket". Vị trí lưu được quyết định bởi hàm băm (Hash). Khi 2 phần tử có cùng mã băm (Collision), chúng tạo thành LinkedList ở cùng 1 bucket. Từ Java 8, nếu LinkedList dài \> 8, nó tự biến thành Red-Black Tree để tăng tốc độ tìm kiếm từ ![][image1] lên ![][image2].
* **Contract của equals() và hashCode():** Nếu 2 object được coi là "bằng nhau" về mặt logic (vd: cùng ID), thì hashCode() BẮT BUỘC phải trả về cùng 1 số. Nếu vi phạm, HashSet sẽ lưu trùng lặp 2 object giống hệt nhau.

## **3.2 Design Patterns trong thực tế**

 

* **Factory Pattern:** Thay vì viết các chuỗi if-else dài dằng dặc ở tầng Controller để quyết định xem user muốn export file PDF, Excel hay CSV, em đưa logic đó vào ExportFactory. Tầng gọi (Client) chỉ cần truyền vào tham số "PDF", Factory sẽ lo việc dùng lệnh new để tạo đúng Object.
* **Observer Pattern (Publish-Subscribe):** Rất phù hợp với kiến trúc Event-driven. Khi một Đơn hàng thanh toán xong, hệ thống cần gửi Email, trừ Tồn kho, tích Điểm thưởng. Thay vì class Order gọi trực tiếp 3 class kia (gây tight-coupling), Order chỉ làm nhiệm vụ "thông báo" (Notify). Các service kia đóng vai trò Observer, tự động lắng nghe và xử lý.

## **3.3 Unit Testing & Mockito**

Unit Test giúp chứng minh code chạy đúng trước khi deploy.

* Vấn đề lớn nhất khi test tầng Service là nó thường gọi xuống tầng Repository (chọc vào DB). Nếu chọc vào DB thật, test sẽ chạy chậm và vi phạm nguyên lý **Isolated** (Độc lập).
* **Giải pháp:** Sử dụng thư viện **Mockito**. Ta sẽ tạo ra các đối tượng giả mạo (Mock) cho tầng DB, ép nó trả về dữ liệu chuẩn bị sẵn. Nhờ đó, em có thể tập trung 100% vào việc test logic if-else của tầng Service.

# **4\. Practice**

 

[Level 1]

* Xử lý Hash Collision: Tạo class Student, Override đúng chuẩn equals() và hashCode() dựa trên id học sinh để bảo vệ tính toàn vẹn dữ liệu khi đưa vào HashSet.  
  @Override  
  public boolean equals(Object o) {  
  if (this == o) return true;  
  if (o == null || getClass() != o.getClass()) return false;  
  Student student = (Student) o;  
  return Objects.equals(id, student.id); // Logic: 2 học sinh là 1 nếu trùng ID  
  }

  @Override  
  public int hashCode() {  
  return Objects.hash(id); // Mã băm sinh ra từ ID  
  }

[Level2]

* Observer Pattern: Thiết kế luồng xử lý "Thanh toán thành công" (Subject) sẽ kích hoạt đồng loạt EmailService và InventoryService (Observers) mà không bị phụ thuộc code lẫn nhau.  
  class Order {  
  private List<OrderObserver> observers = new ArrayList<>();

      // Đăng ký các module muốn nhận thông báo  
      public void addObserver(OrderObserver observer) { observers.add(observer); }

      public void payOrder() {  
          System.out.println("Thanh toán thành công\!");  
          // Báo tin cho tất cả các service đang lắng nghe (Loose coupling)  
          for (OrderObserver obs : observers) {  
              obs.update(this.orderId);  
          }  
      }  
  }

[Level3]

* Viết Unit Test với Mockito: Code test cho AuthService. Thay vì gọi database thật, em dùng @Mock để giả lập UserRepository và dạy nó cách trả về kết quả bằng Mockito.when().  
  @ExtendWith(MockitoExtension.class)  
  public class AuthServiceTest {  
  @Mock  
  UserRepository mockRepo; // Kho dữ liệu giả mạo

      @InjectMocks  
      AuthService authService; // Service thật cần test

      @Test  
      public void testIsAdmin\_Success() {  
          // Arrange: "Dạy" Mock object trả về ADMIN khi truyền ID 123  
          Mockito.when(mockRepo.getRoleById("123")).thenReturn("ADMIN");

          // Act: Gọi hàm nghiệp vụ thực tế  
          boolean result \= authService.isAdmin("123");

          // Assert: Đảm bảo logic xử lý đúng  
          assertTrue(result, "User 123 phải có quyền ADMIN");  
      }  
  }

# **5\. Reflection and Difficulties**

Trong Tuần 2 này, rào cản lớn nhất của em là việc làm quen với tư duy **Mocking** khi viết Unit Test. Ban đầu, cú pháp when(...).thenReturn(...) của Mockito tạo cảm giác khá "ngược đời", và việc xác định ranh giới: "Cái gì cần Mock (làm giả)? Cái gì cần gọi thật?" khiến em lúng túng.

Tuy nhiên, bằng cách áp dụng quy tắc: *"Test class nào thì class đó là thật, toàn bộ các dependency (như Repository, External API) truyền vào nó đều phải là Mock"*, em đã vượt qua được khó khăn này và thấy rõ giá trị của Unit Test trong việc cô lập lỗi (isolate bugs).

Bên cạnh đó, việc học các Design Patterns đôi khi mang lại cảm giác "Over-engineering" (thiết kế quá mức cần thiết) nếu áp dụng cho bài toán nhỏ. Em rút ra kinh nghiệm là chỉ nên dùng Pattern khi thực sự thấy code có dấu hiệu lặp lại hoặc khó mở rộng, tuân theo nguyên lý **KISS** (Keep It Simple, Stupid).

