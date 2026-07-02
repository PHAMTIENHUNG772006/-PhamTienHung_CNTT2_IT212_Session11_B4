# Bài 4: Chiến Lược Sinh Dữ Liệu Giả Lập & Test Case

## 1. Mega-Prompt

```text
Bạn là một Senior Java Backend Developer, Software Test Architect và QA Automation Engineer có hơn 15 năm kinh nghiệm phát triển hệ thống Enterprise.

## Mục tiêu

Sinh đồng thời bộ dữ liệu giả lập (Mock Data) và mã nguồn Unit Test cho chức năng đăng nhập của hệ thống.

## Ngữ cảnh

Hệ thống có phương thức:

```java
login(String username, String password)
```

Hiện tại chưa có source code của phương thức này và cũng chưa có Database test.

Hãy tự giả định logic nghiệp vụ phổ biến của một hệ thống xác thực người dùng để xây dựng dữ liệu kiểm thử và Unit Test.

## Giả định nghiệp vụ

- Nếu username và password đúng → Login thành công.
- Nếu password sai → Login thất bại.
- Nếu tài khoản bị khóa → Login thất bại.
- Nếu username chứa chuỗi SQL Injection → Login thất bại.
- Nếu password rỗng → Login thất bại.

## Yêu cầu

### Phần 1

Sinh Mock Data ở định dạng JSON gồm đúng 5 user đại diện cho các trường hợp:

1. Đăng nhập thành công
2. Sai mật khẩu
3. Tài khoản bị khóa
4. SQL Injection
5. Password rỗng

Mỗi object gồm:

- username
- password
- accountLocked
- expectedResult
- description

### Phần 2

Sinh mã nguồn Java sử dụng:

- Java 17
- JUnit 5

Yêu cầu:

- Tự giả định lớp UserAuthentication.
- Đọc dữ liệu JSON.
- Duyệt toàn bộ test data.
- Thực hiện login().
- So sánh kết quả thực tế với expectedResult bằng Assertions.
- Viết code theo chuẩn Clean Code.
- Thêm comment giải thích các phần quan trọng.

## Ràng buộc

- Không sử dụng Database.
- Không sử dụng Spring Boot.
- Không sử dụng Mockito.
- Mock Data phải được đọc từ JSON.
- Code phải có khả năng mở rộng.

## Định dạng đầu ra

Trả lời theo đúng thứ tự:

1. Mock Data JSON
2. Mã nguồn Java JUnit 5

Không giải thích thêm.
```

---

# 2. Mock Data (JSON)

```json
[
  {
    "username": "john.doe",
    "password": "Password123",
    "accountLocked": false,
    "expectedResult": true,
    "description": "Valid username and password"
  },
  {
    "username": "john.doe",
    "password": "WrongPassword",
    "accountLocked": false,
    "expectedResult": false,
    "description": "Incorrect password"
  },
  {
    "username": "locked.user",
    "password": "Password123",
    "accountLocked": true,
    "expectedResult": false,
    "description": "Locked account"
  },
  {
    "username": "' OR '1'='1",
    "password": "Password123",
    "accountLocked": false,
    "expectedResult": false,
    "description": "SQL Injection attempt"
  },
  {
    "username": "john.doe",
    "password": "",
    "accountLocked": false,
    "expectedResult": false,
    "description": "Empty password"
  }
]
```

---

# 3. JUnit 5 Test Case

```java
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;

import java.io.InputStream;
import java.util.List;

class UserAuthenticationTest {

    /**
     * Model đại diện cho từng test case trong file JSON.
     */
    static class LoginTestData {
        public String username;
        public String password;
        public boolean accountLocked;
        public boolean expectedResult;
        public String description;
    }

    /**
     * Lớp giả lập chức năng xác thực người dùng.
     */
    static class UserAuthentication {

        public boolean login(String username, String password, boolean accountLocked) {

            // Kiểm tra tài khoản bị khóa
            if (accountLocked) {
                return false;
            }

            // Kiểm tra SQL Injection đơn giản
            if (username.contains("'") || username.contains("--")) {
                return false;
            }

            // Kiểm tra mật khẩu rỗng
            if (password == null || password.isEmpty()) {
                return false;
            }

            // Giả lập tài khoản hợp lệ
            return username.equals("john.doe")
                    && password.equals("Password123");
        }
    }

    @Test
    void testLoginUsingMockData() throws Exception {

        ObjectMapper mapper = new ObjectMapper();

        InputStream inputStream = getClass()
                .getResourceAsStream("/login-test-data.json");

        List<LoginTestData> testData =
                mapper.readValue(
                        inputStream,
                        new TypeReference<List<LoginTestData>>() {
                        });

        UserAuthentication authentication = new UserAuthentication();

        for (LoginTestData data : testData) {

            boolean actualResult = authentication.login(
                    data.username,
                    data.password,
                    data.accountLocked
            );

            Assertions.assertEquals(
                    data.expectedResult,
                    actualResult,
                    data.description
            );
        }
    }
}
```

---

# 4. Kết luận

Mega-Prompt trên áp dụng đầy đủ **5 thành phần của Prompt Engineering**:

- **Role (Vai trò):** Senior Java Backend Developer, Software Test Architect và QA Automation Engineer.
- **Task (Mục tiêu):** Sinh đồng thời Mock Data và mã nguồn Unit Test.
- **Context (Ngữ cảnh):** Chức năng `login(String username, String password)` chưa có Database và source code, AI phải tự giả định nghiệp vụ.
- **Constraints (Ràng buộc):** Sử dụng Java 17, JUnit 5, không dùng Database, Spring Boot hay Mockito; Mock Data đọc từ JSON.
- **Output Format (Định dạng):** Trả về theo đúng thứ tự gồm Mock Data JSON và mã nguồn Java JUnit 5, giúp dễ dàng tích hợp vào dự án.