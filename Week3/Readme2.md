# **Week 3 Report**

**Intern:** Nguyễn Như Thuật

**Team:** Platform - Adtech

**Duration:** Week 3

# **1\. Overview**

Trong Tuần 3, em bắt đầu chuyển dịch từ nền tảng Java Core sang việc ứng dụng Framework với **Spring Boot** và **Spring Data JPA (Hibernate)**.

Mục tiêu của tuần này không chỉ là học cách "gọi API cho chạy được", mà là hiểu sâu cơ chế "phép thuật" bên dưới của Spring: **Inversion of Control (IoC)**, **Dependency Injection (DI)** và các **Bean Scope**. Em cũng trực tiếp xây dựng các RESTful API chuẩn mực, áp dụng **DTO Pattern** để bảo vệ cấu trúc Database, và thiết kế **Global Exception Handler** dựa trên tư tưởng AOP để chuẩn hóa luồng lỗi. Đặc biệt, với hệ thống yêu cầu hiệu năng cao như Adtech Platform, em đã dành thời gian mổ xẻ vấn đề hiệu năng kinh điển của ORM: **N+1 Query Problem** (bắt nguồn từ cơ chế Lazy Loading & Proxy) và cách khắc phục.

Qua tuần này, em nhận ra Spring Boot thực chất là một tập hợp các Design Patterns (đã học ở Tuần 2\) được gói gọn lại một cách tinh tế để giúp Developer tập trung hoàn toàn vào Business Logic.

# **2\. Learning Summary**

| Chủ đề             | Đầu mục                                              | Kiến thức đã thu thập                                                                                                                           | Trạng thái       |
|:-------------------|:-----------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------|:-----------------|
| Spring Core        | IoC Container, DI, Bean Scope, Constructor Injection | Hiểu cơ chế Spring tự động quản lý vòng đời object (Bean). Phân biệt Singleton vs Prototype. Hiểu lý do Constructor Injection là Best Practice. | Đã hiểu          |
| Spring Web         | RESTful API, 3-Tier Architecture, DTO Pattern        | Nắm được mô hình Controller-Service-Repository. Bắt buộc dùng DTO làm "màng lọc" để không lộ Entity Database trực tiếp ra API.                  | Đã thực code API |
| Exception Handling | @RestControllerAdvice, AOP (Aspect-Oriented)         | Hiểu tư tưởng AOP đằng sau Advice: tạo "tấm khiên" gom toàn bộ exception từ Service ném ra để map về HTTP Status Code chuẩn (400, 404, 500).    | Đã thực hành     |
| Spring Data JPA    | ORM, Entity, Proxy Object, Lazy Loading              | Hiểu cách Hibernate map Object-Table. Nắm vững khái niệm Proxy Object (Object giả mạo) mà Hibernate dùng để tối ưu RAM (Lazy Loading).          | Đã hiểu          |
| Performance        | N+1 Query Problem, Fetch Join                        | Phát hiện lỗi truy vấn kinh điển làm sập DB khi vòng lặp kích hoạt Proxy liên tục. Biết cách dùng JOIN FETCH để gom thành 1 query.              | Đã hiểu          |

# **3\. Learning Details**

## **3.1 Spring Core (IoC, DI & Bean Scope)**

Tuần trước em đã học Factory Pattern để tránh dùng từ khóa new. Tuần này, em nhận ra **Spring IoC Container** chính là một "siêu Factory".

* **Bean Scope:** Em tìm hiểu sâu về phạm vi tồn tại của Bean. Mặc định Spring dùng **Singleton** (tạo 1 object duy nhất dùng chung), cực kỳ tối ưu bộ nhớ và lý tưởng cho các class không chứa trạng thái thay đổi (Stateless) như Controller hay Service.
* **Constructor Injection vs Field Injection:** Em nhận ra việc dùng @Autowired trực tiếp lên biến (Field Injection) bị coi là "Bad Practice". Việc chuyển sang dùng **Constructor Injection** giúp em có thể đánh dấu dependency là final (bất biến), bảo vệ sự toàn vẹn của class và đặc biệt là cực kỳ dễ dàng khi cần truyền Mock Object vào để viết Unit Test.

## **3.2 Spring Web, DTO Pattern & Tư tưởng AOP**

Em đã làm quen với kiến trúc **Controller \-\> Service \-\> Repository**:

* **DTO Pattern:** Em nhận ra việc expose (phơi bày) trực tiếp class @Entity ra API là cực kỳ nguy hiểm. Nó có thể làm lộ cột nhạy cảm (password, role) và gây lỗi đệ quy vô hạn (Infinite Recursion) khi JSON parser cố dịch các Entity có quan hệ 2 chiều. Mọi dữ liệu trả về đều phải được copy sang DTO chuyên biệt.
* **Global Exception Handler & AOP:** Thay vì viết try-catch rải rác làm rác code Controller, em áp dụng @RestControllerAdvice. Cấu trúc này ứng dụng tư tưởng **AOP (Aspect-Oriented Programming \- Lập trình hướng khía cạnh)**. Nó tạo ra một "tấm khiên" chặn mọi request/response, tự động "bắt" các Exception do tầng Service ném ra giữa không trung và chuyển hóa thành JSON báo lỗi định dạng chuẩn trước khi trả về Client.

## **3.3 Spring Data JPA & Bản chất của N+1 Query (Proxy & Lazy Loading)**

JPA giúp lập trình viên không phải viết SQL thuần, nhưng nếu không hiểu Internals của nó thì rất dễ làm sập hệ thống dưới tải cao.

* **Lazy Loading & Proxy Object:** Để tiết kiệm RAM, khi query 1 User, Hibernate không vội vàng query luôn danh sách 100 bài viết (Articles) của User đó. Nó nhét vào đó một **Proxy Object** (danh sách giả). Chỉ khi code thực sự gọi user.getArticles(), SQL mới được bắn xuống DB.
* **Vấn đề N+1:** N+1 Query sinh ra chính từ kẽ hở của Lazy Loading. Lệnh findAll() lấy ra 10 User (1 query). Khi code lặp qua 10 user này để lấy danh sách bài viết, Proxy bị kích hoạt 10 lần, sinh ra thêm 10 queries nữa (1 \+ 10 \= 11 queries). Nếu lấy 10,000 users, DB sẽ nghẽn mạch vì phải chịu 10,001 queries liên tục.
* **Giải pháp:** Em đã học cách dùng JOIN FETCH trong JPQL để "dẹp" cái Proxy đi, ép Hibernate bắn đúng 1 câu lệnh SQL JOIN duy nhất ngay từ đầu.

# **4\. Practice**

Để chứng minh các khái niệm trên, em đã xây dựng một module API quản lý Chiến dịch Quảng cáo (Campaign) áp dụng đầy đủ các kỹ thuật:

**\[Level 1\] RESTful API & DTO Pattern:**

Xây dựng API tạo mới Campaign. Thay vì nhận Entity, Controller chỉ nhận CampaignCreationDTO và áp dụng Constructor Injection chuẩn mực.

@RestController  
@RequestMapping("/api/v1/campaigns")  
public class CampaignController {

    // Áp dụng Dependency Injection qua Constructor (Best Practice), dependency là final  
    private final CampaignService campaignService;  
    public CampaignController(CampaignService campaignService) {  
        this.campaignService \= campaignService;  
    }

    @PostMapping  
    public ResponseEntity\<CampaignResponseDTO\> createCampaign(@RequestBody CampaignCreationDTO dto) {  
        // Tầng Controller chỉ điều phối, logic tính toán nằm ở Service  
        CampaignResponseDTO result \= campaignService.create(dto);  
        return ResponseEntity.status(HttpStatus.CREATED).body(result);  
    }  
}

**\[Level 2\] Global Exception Handling (AOP):**

Xử lý triệt để các lỗi CustomException tập trung tại một nơi, giúp code Controller sạch sẽ hoàn toàn khỏi các khối try-catch.

@RestControllerAdvice  
public class GlobalExceptionHandler {

    // Lưới lọc bắt toàn bộ lỗi nghiệp vụ không tìm thấy dữ liệu  
    @ExceptionHandler(ResourceNotFoundException.class)  
    public ResponseEntity\<ErrorResponse\> handleNotFound(ResourceNotFoundException ex) {  
        ErrorResponse error \= new ErrorResponse("NOT\_FOUND", ex.getMessage());  
        return ResponseEntity.status(HttpStatus.NOT\_FOUND).body(error);  
    }

    // Lưới lọc bắt toàn bộ lỗi logic/validate  
    @ExceptionHandler(IllegalArgumentException.class)  
    public ResponseEntity\<ErrorResponse\> handleBadRequest(IllegalArgumentException ex) {  
        ErrorResponse error \= new ErrorResponse("BAD\_REQUEST", ex.getMessage());  
        return ResponseEntity.status(HttpStatus.BAD\_REQUEST).body(error);  
    }  
}

**\[Level 3\] Fix lỗi N+1 Query trong Spring Data JPA:**

Em đã phát hiện và xử lý lỗi N+1 khi query danh sách Campaign kèm theo danh sách AdGroups (Quan hệ OneToMany).

public interface CampaignRepository extends JpaRepository\<Campaign, Long\> {

    // CÁCH CŨ (Gây N+1 Query khi lặp qua danh sách Campaign để lấy AdGroups):   
    // List\<Campaign\> findAll();

    // CÁCH TỐI ƯU (Dùng JPQL JOIN FETCH):  
    // Ép Hibernate bỏ qua Proxy, bắn đúng 1 câu SQL JOIN 2 bảng mang hết data lên bộ nhớ  
    @Query("SELECT c FROM Campaign c JOIN FETCH c.adGroups")  
    List\<Campaign\> findAllWithAdGroups();  
}

# **5\. Reflection and Difficulties**

Sự chuyển đổi từ việc tự viết mọi thứ (Java thuần) sang việc giao quyền kiểm soát cho Framework (Spring IoC) mang lại cảm giác khá "lạ lẫm".  Các annotation như @Autowired, @Transactional làm mọi thứ chạy ngầm rất hoàn hảo, nhưng khi có bug xảy ra (ví dụ: LazyInitializationException khi query JPA), em bị bối rối vì không thấy code lỗi một cách rõ ràng.

Tuy nhiên, bằng cách xem kỹ Log SQL do Hibernate sinh ra (spring.jpa.show-sql=true), kết hợp với việc hiểu sâu về khái niệm **Proxy Object**, em đã "bắt bệnh" được các truy vấn chạy ngầm và hiểu rõ hơn về vòng đời của 1 Hibernate Session. Em nhận ra rằng: **Không bao giờ được tin tưởng hoàn toàn vào sự tự động của ORM mà không kiểm tra log query.**

 