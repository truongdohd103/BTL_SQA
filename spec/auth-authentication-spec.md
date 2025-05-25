# Đặc tả kỹ thuật: Module Đăng ký, Đăng nhập, Xác thực và Phân quyền

## 1. Tổng quan

Module Đăng ký, Đăng nhập, Xác thực và Phân quyền là thành phần nền tảng của hệ thống thương mại điện tử, cung cấp các chức năng:
- Đăng ký tài khoản người dùng mới
- Xác thực email thông qua OTP
- Đăng nhập vào hệ thống
- Đăng xuất khỏi hệ thống
- Đổi mật khẩu
- Phân quyền người dùng (user/admin)

### 1.1. Mục tiêu
- Cung cấp quy trình đăng ký và đăng nhập an toàn, dễ sử dụng
- Xác thực người dùng thông qua email
- Bảo vệ thông tin người dùng
- Phân quyền truy cập vào các chức năng của hệ thống

### 1.2. Công nghệ sử dụng
- **Backend**: NestJS, TypeORM, MySQL
- **Frontend**: ReactJS, TailwindCSS
- **API**: RESTful API
- **Xác thực**: JWT, OTP
- **Mã hóa**: bcrypt
- **Gửi email**: nodemailer

## 2. Đăng ký tài khoản

### 2.1. Đăng ký người dùng mới

- **Mô tả**: Đăng ký tài khoản người dùng mới và gửi OTP xác thực qua email
- **API**: `POST /register`
- **Controller**: `RegisterModuleController.create()`
- **Service**: `RegisterModuleService.create()`
- **Repository**: `UserRepository.save()`
- **Frontend**: 
  - Component: `RegisterForm.jsx`
  - Service: `registerUser(formData)` trong `auth-api.js`
- **Dữ liệu gửi lên** (`CreateUserDto`):
```json
{
  "firstName": "Nguyễn",
  "lastName": "Văn A",
  "email": "nguyenvana@example.com",
  "password": "password123",
  "phone": "0123456789",
  "address": "123 Đường ABC, Quận XYZ, TP.HCM"
}
```
- **Validation**:
  - Họ, tên: bắt buộc
  - Email: bắt buộc, đúng định dạng email
  - Mật khẩu: bắt buộc, độ dài 8-24 ký tự
  - Số điện thoại: bắt buộc, đúng định dạng số điện thoại Việt Nam
  - Địa chỉ: bắt buộc
- **Xử lý**:
  - Kiểm tra email đã tồn tại chưa
  - Nếu đã tồn tại và đã kích hoạt: báo lỗi
  - Nếu đã tồn tại nhưng chưa kích hoạt: gửi lại OTP
  - Nếu chưa tồn tại: tạo tài khoản mới, lưu thông tin địa chỉ, gửi OTP
- **Dữ liệu trả về**: Email đã đăng ký

### 2.2. Xác thực OTP

- **Mô tả**: Xác thực tài khoản thông qua mã OTP gửi qua email
- **API**: `PATCH /register`
- **Controller**: `RegisterModuleController.update()`
- **Service**: `RegisterModuleService.update()`
- **Repository**: `UserRepository.update()`
- **Frontend**: 
  - Component: `OtpVerificationForm.jsx`
  - Service: `verifyOtp(email, otp)` trong `auth-api.js`
- **Dữ liệu gửi lên** (`VerifyDto`):
```json
{
  "email": "nguyenvana@example.com",
  "otp": "123456"
}
```
- **Validation**:
  - Email: bắt buộc, đúng định dạng email
  - OTP: bắt buộc, 6 chữ số
- **Xử lý**:
  - Kiểm tra OTP có hợp lệ không (sử dụng thư viện otplib)
  - Nếu hợp lệ: cập nhật trạng thái tài khoản thành kích hoạt (isActive = true)
  - Nếu không hợp lệ hoặc hết hạn: báo lỗi
- **Dữ liệu trả về**: Kết quả xác thực (true/false)

## 3. Đăng nhập

- **Mô tả**: Đăng nhập vào hệ thống và nhận JWT token
- **API**: `POST /login`
- **Controller**: `LoginModuleController.login()`
- **Service**: `LoginModuleService.login()`
- **Repository**: `UserRepository.findOneBy()`
- **Frontend**: 
  - Component: `LoginForm.jsx`
  - Service: `loginApi(credentials)` trong `auth-api.js`
- **Dữ liệu gửi lên** (`LoginDTO`):
```json
{
  "email": "nguyenvana@example.com",
  "password": "password123"
}
```
- **Validation**:
  - Email: bắt buộc, đúng định dạng email
  - Mật khẩu: bắt buộc
- **Xử lý**:
  - Kiểm tra email có tồn tại không
  - So sánh mật khẩu đã mã hóa
  - Tạo JWT token với thông tin người dùng (id, email, role)
  - Lưu token vào database
- **Dữ liệu trả về**:
```json
{
  "user": {
    "id": "uuid-string",
    "email": "nguyenvana@example.com",
    "firstName": "Nguyễn",
    "lastName": "Văn A",
    "role": "user"
  },
  "accessToken": "jwt-token-string"
}
```

## 4. Đăng xuất

- **Mô tả**: Đăng xuất khỏi hệ thống
- **API**: `POST /logout/:user_id`
- **Controller**: `LogoutController.logout()`
- **Service**: `LogoutService.logout()`
- **Repository**: `UserRepository.save()`
- **Frontend**: 
  - Component: `LogoutButton.jsx`
  - Service: `logoutUser()` trong `auth-api.js`
- **Quyền**: Người dùng đã đăng nhập
- **Dữ liệu gửi lên** (`logoutDTO`):
```json
{
  "token": "jwt-token-string"
}
```
- **Xử lý**:
  - Kiểm tra người dùng và token có hợp lệ không
  - Xóa token trong database (đặt token = null)
- **Dữ liệu trả về**: Kết quả đăng xuất (true/false)

## 5. Đổi mật khẩu

- **Mô tả**: Thay đổi mật khẩu người dùng
- **API**: `POST /change-password/:user_id`
- **Controller**: `ChangePasswordController.changePassword()`
- **Service**: `ChangePasswordService.changePassword()`
- **Repository**: `UserRepository.save()`
- **Frontend**: 
  - Component: `ChangePasswordForm.jsx`
  - Service: `changePassword(userId, passwordData)` trong `auth-api.js`
- **Quyền**: Người dùng đã đăng nhập
- **Dữ liệu gửi lên** (`changePassDTO`):
```json
{
  "password": "old-password",
  "newPassword": "new-password",
  "newPasswordCf": "new-password"
}
```
- **Validation**:
  - Mật khẩu cũ: bắt buộc
  - Mật khẩu mới: bắt buộc
  - Xác nhận mật khẩu mới: phải khớp với mật khẩu mới
- **Xử lý**:
  - Kiểm tra mật khẩu cũ có đúng không
  - Mã hóa mật khẩu mới
  - Cập nhật mật khẩu trong database
- **Dữ liệu trả về**: Kết quả đổi mật khẩu (true/false)

## 6. Xác thực và Phân quyền

### 6.1. JWT Authentication Guard

- **Mô tả**: Middleware kiểm tra JWT token trong mọi request yêu cầu xác thực
- **File**: `JwtAuth.guard.ts`
- **Xử lý**:
  - Trích xuất token từ header Authorization
  - Xác thực token bằng JwtService
  - Kiểm tra token có tồn tại trong database không
  - Đính kèm thông tin người dùng vào request

### 6.2. Roles Guard

- **Mô tả**: Middleware kiểm tra quyền của người dùng
- **File**: `Roles.guard.ts`
- **Decorator**: `@Roles('admin', 'user')`
- **Xử lý**:
  - Lấy thông tin người dùng từ request
  - Kiểm tra role của người dùng có phù hợp với yêu cầu không
  - Cho phép hoặc từ chối truy cập

## 7. Cấu trúc dữ liệu

### 7.1. Người dùng (`User`)

```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ unique: true })
  email: string;

  @Column()
  password: string;

  @Column({ default: 'user' })
  role: string;

  @Column({ nullable: true })
  token: string;

  @Column({ default: false })
  isActive: boolean;

  @OneToMany(() => Location_userEntity, (location) => location.user)
  locations: Location_userEntity[];
}
```

### 7.2. Địa chỉ người dùng (`Location_userEntity`)

```typescript
@Entity()
export class Location_userEntity {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  name: string;

  @Column()
  address: string;

  @Column()
  phone: string;

  @Column({ default: false })
  default_location: boolean;

  @Column()
  user_id: string;

  @ManyToOne(() => User, (user) => user.locations)
  user: User;
}
```

## 8. Giao diện người dùng

### 8.1. Trang đăng ký

- **Component**: `RegisterPage.jsx`
- **Chức năng**:
  - Form đăng ký với các trường thông tin cần thiết
  - Validation form
  - Hiển thị thông báo lỗi
  - Chuyển hướng đến trang xác thực OTP sau khi đăng ký thành công

### 8.2. Trang xác thực OTP

- **Component**: `OtpVerificationPage.jsx`
- **Chức năng**:
  - Nhập mã OTP nhận được qua email
  - Hiển thị thời gian hết hạn OTP
  - Nút gửi lại OTP
  - Chuyển hướng đến trang đăng nhập sau khi xác thực thành công

### 8.3. Trang đăng nhập

- **Component**: `LoginPage.jsx`
- **Chức năng**:
  - Form đăng nhập với email và mật khẩu
  - Validation form
  - Hiển thị thông báo lỗi
  - Lưu token vào localStorage sau khi đăng nhập thành công
  - Chuyển hướng đến trang chủ sau khi đăng nhập thành công

### 8.4. Trang đổi mật khẩu

- **Component**: `ChangePasswordPage.jsx`
- **Chức năng**:
  - Form đổi mật khẩu với các trường: mật khẩu cũ, mật khẩu mới, xác nhận mật khẩu mới
  - Validation form
  - Hiển thị thông báo thành công/lỗi

## 9. Xử lý lỗi

- **Validation**: Sử dụng class-validator để kiểm tra dữ liệu đầu vào
- **Exception handling**: Xử lý các trường hợp lỗi và trả về thông báo phù hợp
- **Response format chuẩn** cho lỗi:
```json
{
  "success": false,
  "status": 400,
  "message": "LOGIN.USER.EMAIL IS NOT VALID!",
  "errors": []
}
```

## 10. Bảo mật

- **Mã hóa mật khẩu**: Sử dụng bcrypt với salt 10
- **JWT**: Sử dụng JWT để xác thực người dùng
- **OTP**: Sử dụng otplib để tạo và xác thực OTP, thời gian hết hạn 2 phút
- **CORS**: Cấu hình CORS để cho phép truy cập từ frontend