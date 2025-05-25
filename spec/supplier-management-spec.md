# Đặc tả kỹ thuật: Module Quản lý Nhà cung cấp

## 1. Tổng quan

Module Quản lý Nhà cung cấp là thành phần quan trọng trong hệ thống thương mại điện tử, cho phép:
- Quản trị viên: thêm, sửa, xóa và quản lý danh sách nhà cung cấp
- Liên kết nhà cung cấp với sản phẩm trong hệ thống

### 1.1. Mục tiêu
- Cung cấp công cụ quản lý hiệu quả cho quản trị viên
- Theo dõi thông tin nhà cung cấp
- Hỗ trợ tìm kiếm và lọc nhà cung cấp nhanh chóng

### 1.2. Công nghệ sử dụng
- **Backend**: NestJS, TypeORM, MySQL
- **Frontend**: ReactJS, TailwindCSS
- **API**: RESTful API

## 2. Quản lý nhà cung cấp

### 2.1. Danh sách nhà cung cấp

- **Mô tả**: Hiển thị danh sách nhà cung cấp với phân trang
- **API**: `GET /supplier/:page/:limit`
- **Controller**: `SupplierController.getList()`
- **Service**: `SupplierService.getList()`
- **Repository**: `SupplierRepository.findAndCount()`
- **Quyền**: Chỉ admin mới có quyền thực hiện (`@Roles('admin')`)
- **Tham số**:
  - `page`: Số trang
  - `limit`: Số nhà cung cấp trên mỗi trang
- **Dữ liệu trả về**:
  - Danh sách nhà cung cấp
  - Tổng số nhà cung cấp
  - Thông tin phân trang

### 2.2. Thêm nhà cung cấp

- **Mô tả**: Thêm nhà cung cấp mới vào hệ thống
- **API**: `POST /supplier`
- **Controller**: `SupplierController.create()`
- **Service**: `SupplierService.create()`
- **Repository**: `SupplierRepository.save()`
- **Frontend**: 
  - Component: `AddSupplierForm.jsx`
  - Service: `addSupplier(formData)` trong `supplier-service.js`
- **Quyền**: Chỉ admin mới có quyền thực hiện (`@Roles('admin')`)
- **Dữ liệu gửi lên** (`CreateSupplierDto`):
```json
{
  "name": "Tên nhà cung cấp",
  "url_image": "url-image",
  "phone": "0123456789",
  "address": "Địa chỉ nhà cung cấp"
}
```
- **Validation**:
  - Tên nhà cung cấp: bắt buộc
  - URL hình ảnh: bắt buộc
  - Số điện thoại: bắt buộc, đúng định dạng số điện thoại Việt Nam
  - Địa chỉ: bắt buộc
- **Dữ liệu trả về**: Thông tin nhà cung cấp đã tạo
- **Giao diện**:
  - Form với các trường nhập liệu
  - Upload hình ảnh với preview
  - Validation form

### 2.3. Chi tiết nhà cung cấp

- **Mô tả**: Xem chi tiết thông tin nhà cung cấp
- **API**: `GET /supplier/:id`
- **Controller**: `SupplierController.findOne()`
- **Service**: `SupplierService.detail()`
- **Repository**: `SupplierRepository.findOne()`
- **Quyền**: Chỉ admin mới có quyền thực hiện (`@Roles('admin')`)
- **Tham số**: `id`: ID nhà cung cấp
- **Dữ liệu trả về**: Thông tin chi tiết nhà cung cấp

### 2.4. Cập nhật nhà cung cấp

- **Mô tả**: Cập nhật thông tin nhà cung cấp
- **API**: `PATCH /supplier/:id`
- **Controller**: `SupplierController.update()`
- **Service**: `SupplierService.update()`
- **Repository**: `SupplierRepository.update()`
- **Frontend**: 
  - Component: `EditSupplierForm.jsx`
  - Service: `updateSupplier(id, formData)` trong `supplier-service.js`
- **Quyền**: Chỉ admin mới có quyền thực hiện (`@Roles('admin')`)
- **Dữ liệu gửi lên** (`UpdateSupplierDto`):
```json
{
  "id": "uuid-string",
  "name": "Tên nhà cung cấp đã cập nhật",
  "url_image": "url-image-updated",
  "phone": "0123456789",
  "address": "Địa chỉ nhà cung cấp đã cập nhật"
}
```
- **Validation**: Tương tự như khi thêm mới
- **Dữ liệu trả về**: Thông tin nhà cung cấp đã cập nhật

### 2.5. Xóa nhà cung cấp

- **Mô tả**: Xóa nhà cung cấp khỏi hệ thống
- **API**: `DELETE /supplier/:id`
- **Controller**: `SupplierController.remove()`
- **Service**: `SupplierService.delete()`
- **Repository**: `SupplierRepository.delete()`
- **Quyền**: Chỉ admin mới có quyền thực hiện (`@Roles('admin')`)
- **Tham số**: `id`: ID nhà cung cấp
- **Xử lý**:
  - Kiểm tra nhà cung cấp có tồn tại không
  - Kiểm tra nhà cung cấp có đang được sử dụng trong sản phẩm không
  - Xóa nhà cung cấp nếu không có ràng buộc
- **Dữ liệu trả về**: Kết quả xóa

## 3. Tìm kiếm và lọc nhà cung cấp

- **Mô tả**: Tìm kiếm và lọc nhà cung cấp theo các tiêu chí
- **API**: `POST /supplier/search`
- **Controller**: `SupplierController.search()`
- **Service**: `SupplierService.getList()`
- **Repository**: `SupplierRepository.findAndCount()`
- **Quyền**: Chỉ admin mới có quyền thực hiện (`@Roles('admin')`)
- **Dữ liệu gửi lên** (`SearchSupplierDto`):
  - `page`: Số trang
  - `limit`: Số nhà cung cấp trên mỗi trang
  - `name`: Tên nhà cung cấp (tìm kiếm mờ)
  - `phone`: Số điện thoại (tìm kiếm mờ)
- **Dữ liệu trả về**:
  - Danh sách nhà cung cấp phù hợp với điều kiện tìm kiếm
  - Tổng số nhà cung cấp phù hợp
  - Thông tin phân trang

## 4. Cấu trúc dữ liệu

### 4.1. Nhà cung cấp (`SupplierEntity`)

```typescript
@Entity({ name: 'suppliers' })
export class SupplierEntity {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ type: 'text' })
  name: string;

  @Column({ type: 'text' })
  url_image: string;

  @Column({ type: 'text' })
  phone: string;

  @Column({ type: 'text' })
  address: string;

  // Mối quan hệ với ProductEntity
  @OneToMany(() => ProductEntity, (product) => product.supplier)
  products: ProductEntity[]; // Danh sách sản phẩm thuộc về nhà cung cấp
}
```

### 4.2. DTO

#### 4.2.1. Tạo nhà cung cấp (`CreateSupplierDto`)

```typescript
export class CreateSupplierDto {
  @ApiProperty()
  @IsNotEmpty()
  @Expose()
  name: string;

  @ApiProperty()
  @IsNotEmpty()
  @Expose()
  url_image: string;

  @ApiProperty()
  @IsNotEmpty()
  @Expose()
  @IsPhoneNumber('VN')
  phone: string;

  @ApiProperty()
  @IsNotEmpty()
  @Expose()
  address: string;
}
```

#### 4.2.2. Cập nhật nhà cung cấp (`UpdateSupplierDto`)

```typescript
export class UpdateSupplierDto extends PartialType(CreateSupplierDto) {
  @ApiProperty()
  @IsNotEmpty()
  @Expose()
  id: string;
}
```

#### 4.2.3. Tìm kiếm nhà cung cấp (`SearchSupplierDto`)

```typescript
export class SearchSupplierDto {
  @ApiProperty({ required: false })
  @IsOptional()
  @Expose()
  name?: string;

  @ApiProperty({ required: false })
  @IsOptional()
  @Expose()
  phone?: string;

  @ApiProperty()
  @IsNotEmpty()
  @Expose()
  page: number;

  @ApiProperty()
  @IsNotEmpty()
  @Expose()
  limit: number;
}
```

## 5. Giao diện người dùng

### 5.1. Trang quản lý nhà cung cấp

- **Component**: `SupplierManagement.jsx`
- **Chức năng**:
  - Hiển thị danh sách nhà cung cấp dưới dạng bảng
  - Phân trang
  - Tìm kiếm và lọc
  - Nút thêm, sửa, xóa nhà cung cấp
- **Hiển thị**:
  - Bảng với các cột: ID, Hình ảnh, Tên, Số điện thoại, Địa chỉ, Hành động
  - Form tìm kiếm
  - Phân trang

### 5.2. Form thêm/sửa nhà cung cấp

- **Component**: `SupplierForm.jsx`
- **Chức năng**:
  - Form nhập thông tin nhà cung cấp
  - Upload hình ảnh
  - Validation
- **Hiển thị**:
  - Các trường nhập liệu: Tên, Số điện thoại, Địa chỉ
  - Upload hình ảnh với preview
  - Nút Lưu và Hủy

## 6. Xử lý lỗi

- **Validation**: Sử dụng class-validator để kiểm tra dữ liệu đầu vào
- **Exception handling**: Xử lý các trường hợp lỗi và trả về thông báo phù hợp
- **Response format chuẩn** cho lỗi:
```json
{
  "success": false,
  "status": 400,
  "message": "Failed to create supplier: Invalid phone number",
  "errors": []
}
```

## 7. Bảo mật

- **Phân quyền**: Chỉ admin mới có quyền quản lý nhà cung cấp
- **Guards**:
  - `JwtAuthGuard`: Xác thực người dùng
  - `RolesGuard`: Kiểm tra quyền