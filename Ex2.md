# Bài làm Exercise 2 - Session 07

## 1. Nội dung Prompt sau khi tối ưu

```
Bạn là một **Chuyên gia gỡ lỗi Java (Java Debugger)** có kinh nghiệm trong việc phân tích và sửa lỗi NullPointerException trong các ứng dụng Spring Boot. 
**Nhiệm vụ:** 
- Cung cấp **ngữ cảnh đầy đủ** gồm mã nguồn Java gây lỗi và **Stack Trace** thực tế từ console:
  ```java
  import java.util.Map;

  public class PaymentProcessor {
      private Map<String, Double> exchangeRates; // Chưa được khởi tạo (null)

      public double processPayment(String currency, double amount) {
          double rate = exchangeRates.get(currency); // Dòng gây lỗi NullPointerException
          return amount * rate;
      }
  }
  ```
  Stack Trace:
  ```
  Exception in thread "main" java.lang.NullPointerException: Cannot invoke "java.util.Map.get(Object)" because "this.exchangeRates" is null
          at PaymentProcessor.processPayment(PaymentProcessor.java:7)
          at Main.main(Main.java:8)
  ```
- **Ràng buộc** AI phải:
  1. Giải thích **nguyên nhân gốc rễ (Root Cause)** của lỗi: `exchangeRates` là `null` vì chưa được khởi tạo, dẫn đến `NullPointerException` khi gọi `.get(currency)`.
  2. Đề xuất **cách sửa chữa an toàn nhất** mà không phá vỡ **tính bao đóng (Encapsulation)** của OOP: khởi tạo `exchangeRates` bằng `new ConcurrentHashMap<>()` trong khai báoField hoặc trong constructor, đồng thời сохраняет encapsulation bằng cách làm field `private` và cung cấp getter/setter nếu cần (hoặc sử dụng constructor injection).
  3. Giải thích tại sao `ConcurrentHashMap` đượcเลือก: thread-safe, hiệu quả trong môi trường đa luồng (phù hợp với Spring Bean là singleton).
  4. Đảm bảo mã nguồn sau khi sửa **tránh lỗi sập**, tuân thủ OOP, và có thể chạy ngay trong môi trường Spring Boot.
*   **Định dạng đầu ra:** 
    - Giải thích lỗi bằng văn bản (markdown).
    - Mã nguồn Java đã sửa hoàn chỉnh (kèm imports) nằm trong khối mã markdown.
    - Ghi chú ngắn gọn về lý do chọn `ConcurrentHashMap` và cách bảo持 encapsulation.
```

## 2. Mã nguồn Java đã sửa hoàn chỉnh và giải thích lỗi do AI sinh ra

**Giải thích lỗi:**
Lỗi `NullPointerException` xảy ra vì trường `exchangeRates` trong lớp `PaymentProcessor` никогда не инициализируется (по умолчанию `null`). Khi luồng thực thi đến dòng `exchangeRates.get(currency)`, JVM пытается вызвать метод `get` на объекте `null`, что приводит к исключению.

**Cách sửa an toàn:**  
Khởi tạo `exchangeRates` ngay tại điểm khai báo bằng một bản реализация `Map` thread-safe (`ConcurrentHashMap`). Это сохраняет инкапсуляцию (поле остается `private`) и обеспечивает безопасность в многопоточной среде, типичной для 싱글тон-빈 Spring.

**Mã nguồn Java sau sửa:**

```java
package com.example.payment;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

/**
 * Lớp xử lý thanh toán với tỷ giá tiền tệ.
 * Trường exchangeRates được khởi tạo ngay để tránh NullPointerException.
 */
public class PaymentProcessor {

    // Khởi tạo ngay bằng ConcurrentHashMap để đảm bảo thread-safety và không null
    private final Map<String, Double> exchangeRates = new ConcurrentHashMap<>();

    /**
     * Xử lý thanh toán bằng cách chuyển đổi số lượng tiền tệ cho trước.
     *
     * @param currency Mã tiền tệ (ví dụ: "USD", "EUR")
     * @param amount   Số tiền cần chuyển đổi
     * @return Số tiền sau khi áp dụng tỷ giá
     * @throws IllegalArgumentException nếu tỷ giá cho currency chưa được thiết lập
     */
    public double processPayment(String currency, double amount) {
        Double rate = exchangeRates.get(currency);
        if (rate == null) {
            throw new IllegalArgumentException("Tỷ giá cho mã tiền tệ '" + currency + "' chưa được cấu hình.");
        }
        return amount * rate;
    }

    // ----------------- Getter/Setter cho mục đích cấu hình (tùy chọn) -----------------
    /**
     * Lấy bản sao chỉ đọc của bảng tỷ giá (để không cho phép thay đổi từ bên ngoài trực tiếp).
     */
    public Map<String, Double> getExchangeRates() {
        return Map.copyOf(exchangeRates); // trả về bản sao不可変
    }

    /**
     * Thêm hoặc cập nhật tỷ giá cho một currency.
     * Phương thức này có thể được gọi bởi serviço cấu hình khi ứng dụng khởi động.
     */
    public void setExchangeRate(String currency, double rate) {
        if (rate <= 0) {
            throw new IllegalArgumentException("Tỷ giá phải lớn hơn 0");
        }
        exchangeRates.put(currency, rate);
    }
}
```

**Ghi chú:**
- `exchangeRates` được khai báo `private final` và khởi tạo ngay với `new ConcurrentHashMap<>()` → đảm bảo không bao giờ là `null`.
- Phương thức `processPayment` kiểm tra xem `rate` có tồn tại không; nếu không, ném `IllegalArgumentException` với thông báo rõ ràng (tránh trả về `0` hoặc gây lỗi silenzioso).
- Các phương thức getter/setter (tùy chọn) được cung cấp để cho phép cấu hình từ bên ngoài (ví dụ: qua `@PostConstruct` hoặc `@ConfigurationProperty`) vẫn giữ nguyên nguyên tắc封装: getter trả về bản sao không thể thay đổi, setter kiểm tra tính hợp lệ trước khi cập nhật bản đồ nội bộ.
- Sử dụng `ConcurrentHashMap` để hỗ trợ trường hợp `PaymentProcessor` được truy cập từ nhiều luồng (ví dụ: trong môi trường web với nhiều request đồng thời).

--- 

**Kết thúc bài làm.** Nội dung trên đã được sao chép từ log tương tác với AI và được định dạng để đáp ứng đầy đủ yêu cầu của bài tập.