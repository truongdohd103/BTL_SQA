# Đặc tả kỹ thuật: Module Quản lý Sản phẩm

## 1. Tổng quan

Module Quản lý Sản phẩm là một thành phần quan trọng trong hệ thống thương mại điện tử, cho phép:
- Người dùng: xem, tìm kiếm và lọc sản phẩm
- Quản trị viên: thêm, sửa, xóa và quản lý danh mục sản phẩm

### 1.1. Mục tiêu
- Cung cấp giao diện thân thiện cho người dùng để duyệt và tìm kiếm sản phẩm
- Cung cấp công cụ quản lý hiệu quả cho quản trị viên
- Đảm bảo hiệu suất cao khi xử lý lượng lớn sản phẩm
- Hỗ trợ tìm kiếm và lọc nhanh chóng

### 1.2. Công nghệ sử dụng
- **Backend**: NestJS, TypeORM, MySQL
- **Frontend**: ReactJS, TailwindCSS
- **API**: RESTful API

## 2. Quản lý sản phẩm cho người dùng

### 2.1. Hiển thị sản phẩm

#### 2.1.1. Trang chủ - Sản phẩm nổi bật
- **Mô tả**: Hiển thị các sản phẩm nổi bật trên trang chủ dựa trên doanh thu
- **API**: `GET /dashboard/feature-product`
- **Controller**: `DashboardController.getFeatureProduct()`
- **Service**: `DashboardService.getFeatureProduct()`
- **Repository**: `OrderProductRepository.getFeatureProductsByRevenue()`
- **Frontend**: 
  - Component: `FeatureProducts.jsx`
  - Service: `getFeatureProducts()` trong `product-service.js`
- **Dữ liệu trả về**: 
```json
[
  {
    "productId": "uuid-string",
    "productName": "Tên sản phẩm",
    "productImage": "url-string",
    "priceout": 100000,
    "categoryName": "Tên danh mục",
    "totalRevenue": 5000000
  }
]
```
- **Cách hiển thị**: Grid với tối đa 8 sản phẩm, mỗi sản phẩm hiển thị hình ảnh, tên, giá và danh mục

#### 2.1.2. Trang chủ - Sản phẩm mới nhất
- **Mô tả**: Hiển thị các sản phẩm mới nhất được thêm vào hệ thống
- **API**: `GET /dashboard/latest-product`
- **Controller**: `DashboardController.getLatestProduct()`
- **Service**: `DashboardService.getLatestProduct()`
- **Repository**: `ImportProductRepository.findLatestProducts()`
- **Frontend**: 
  - Component: `LatestProducts.jsx`
  - Service: `getLatestProducts()` trong `product-service.js`
- **Dữ liệu trả về**: 
```json
[
  {
    "id": "uuid-string",
    "name": "Tên sản phẩm",
    "url_images": "json-string-with-image-urls",
    "priceout": 100000,
    "description": "Mô tả sản phẩm",
    "stockQuantity": 50,
    "categoryName": "Tên danh mục"
  }
]
```
- **Cách hiển thị**: Carousel với các sản phẩm mới nhất, mỗi sản phẩm hiển thị hình ảnh, tên, giá và nút "Thêm vào giỏ hàng"

#### 2.1.3. Danh sách sản phẩm
- **Mô tả**: Hiển thị danh sách sản phẩm có phân trang và hỗ trợ lọc
- **API**: `GET /product/:page/:limit`
- **Controller**: `ProductController.getAll()`
- **Service**: `ProductService.getAll()`
- **Repository**: `ProductRepository.findAndCount()`
- **Frontend**: 
  - Component: `ProductList.jsx`
  - Service: `getProducts(page, limit)` trong `product-service.js`
- **Tham số**:
  - `page`: Số trang hiện tại (bắt đầu từ 1)
  - `limit`: Số sản phẩm trên mỗi trang (mặc định: 10)
- **Dữ liệu trả về**: 
```json
{
  "products": [
    {
      "id": "uuid-string",
      "name": "Tên sản phẩm",
      "priceout": 100000,
      "description": "Mô tả sản phẩm",
      "stockQuantity": 50,
      "weight": 500,
      "url_images": "json-string-with-image-urls",
      "category": {
        "id": "uuid-string",
        "name": "Tên danh mục"
      }
    }
  ],
  "total": 100
}
```
- **Cách hiển thị**: 
  - Grid hoặc danh sách tùy theo lựa chọn của người dùng
  - Phân trang ở dưới cùng
  - Mỗi sản phẩm hiển thị hình ảnh, tên, giá, danh mục và nút "Thêm vào giỏ hàng"
  - Khi hover hiển thị nút "Xem chi tiết"

#### 2.1.4. Chi tiết sản phẩm
- **Mô tả**: Hiển thị thông tin chi tiết của một sản phẩm
- **API**: `GET /product/detail/:id`
- **Controller**: `ProductController.detail()`
- **Service**: `ProductService.detail()`
- **Repository**: `ProductRepository.findOne()`
- **Frontend**: 
  - Component: `ProductDetail.jsx`
  - Service: `fetchProductDetail(productId)` trong `product-service.js`
- **Tham số**: `id`: ID của sản phẩm
- **Dữ liệu trả về**: 
```json
{
  "products": {
    "id": "uuid-string",
    "name": "Tên sản phẩm",
    "priceout": 100000,
    "description": "Mô tả chi tiết sản phẩm",
    "stockQuantity": 50,
    "weight": 500,
    "url_images": "{\"url_images1\":\"url1\",\"url_images2\":\"url2\"}",
    "category_id": "uuid-string",
    "supplier_id": "uuid-string",
    "expire_date": "2023-12-31T00:00:00.000Z",
    "category": {
      "id": "uuid-string",
      "name": "Tên danh mục"
    }
  }
}
```
- **Cách hiển thị**: 
  - Layout chia làm 2 phần: hình ảnh bên trái, thông tin bên phải
  - Phần hình ảnh: hình ảnh chính lớn và các hình ảnh nhỏ bên dưới để chọn
  - Phần thông tin: tên sản phẩm, giá, mô tả, số lượng trong kho, danh mục
  - Nút "Thêm vào giỏ hàng" với bộ chọn số lượng
  - Mô tả chi tiết sản phẩm ở phần dưới

### 2.2. Tìm kiếm và lọc sản phẩm

#### 2.2.1. Tìm kiếm theo tên sản phẩm
- **Mô tả**: Tìm kiếm sản phẩm theo tên (hỗ trợ tìm kiếm mờ)
- **API**: `GET /product/search/:page/:limit?name=:name`
- **Controller**: `ProductController.search()`
- **Service**: `ProductService.search()`
- **Repository**: Custom query với `LIKE` operator
- **Frontend**: 
  - Component: `SearchBar.jsx`, `ProductList.jsx`
  - Service: `searchProducts(page, limit, filters)` trong `product-service.js`
- **Tham số**:
  - `page`: Số trang hiện tại
  - `limit`: Số sản phẩm trên mỗi trang
  - `name`: Tên sản phẩm cần tìm kiếm (query parameter)
- **Dữ liệu trả về**: Tương tự như API danh sách sản phẩm
- **Cách hiển thị**: 
  - Thanh tìm kiếm ở header
  - Kết quả hiển thị tương tự như danh sách sản phẩm
  - Hiển thị "Không tìm thấy sản phẩm" nếu không có kết quả

#### 2.2.2. Lọc theo danh mục
- **Mô tả**: Lọc sản phẩm theo danh mục
- **API**: `GET /product/search/:page/:limit?category=:category_id`
- **Controller**: `ProductController.search()`
- **Service**: `ProductService.search()`
- **Repository**: Query với điều kiện `category_id`
- **Frontend**: 
  - Component: `CategoryFilter.jsx`, `ProductList.jsx`
  - Service: `searchProducts(page, limit, filters)` trong `product-service.js`
- **Tham số**:
  - `page`: Số trang hiện tại
  - `limit`: Số sản phẩm trên mỗi trang
  - `category`: ID danh mục cần lọc (query parameter)
- **Dữ liệu trả về**: Tương tự như API danh sách sản phẩm
- **Cách hiển thị**: 
  - Danh sách danh mục ở sidebar
  - Kết quả hiển thị tương tự như danh sách sản phẩm
  - Hiển thị tên danh mục đang được lọc

#### 2.2.3. Sắp xếp sản phẩm
- **Mô tả**: Sắp xếp sản phẩm theo các tiêu chí
- **API**: `GET /product/search/:page/:limit?sort=:sort&order=:order`
- **Controller**: `ProductController.search()`
- **Service**: `ProductService.search()`
- **Repository**: Query với `ORDER BY`
- **Frontend**: 
  - Component: `SortOptions.jsx`, `ProductList.jsx`
  - Service: `searchProducts(page, limit, filters)` trong `product-service.js`
- **Tham số**:
  - `page`: Số trang hiện tại
  - `limit`: Số sản phẩm trên mỗi trang
  - `sort`: Trường cần sắp xếp (name, priceout, createdAt)
  - `order`: Thứ tự sắp xếp (ASC, DESC)
- **Dữ liệu trả về**: Tương tự như API danh sách sản phẩm
- **Cách hiển thị**: 
  - Dropdown cho phép chọn tiêu chí sắp xếp
  - Các tùy chọn: Mới nhất, Giá tăng dần, Giá giảm dần, Tên A-Z, Tên Z-A

## 3. Quản lý sản phẩm cho quản trị viên

### 3.1. Thêm sản phẩm mới

- **Mô tả**: Thêm sản phẩm mới vào hệ thống
- **API**: `POST /product`
- **Controller**: `ProductController.create()`
- **Service**: `ProductService.create()`
- **Repository**: `ProductRepository.save()`
- **Frontend**: 
  - Component: `AddProductForm.jsx`
  - Service: `addProduct(formData)` trong `product-service.js`
- **Quyền**: Chỉ admin mới có quyền thực hiện (`@Roles('admin')`)
- **Dữ liệu gửi lên** (`ProductCreateDTO`):
```json
{
  "name": "Tên sản phẩm",
  "priceout": 100000,
  "banner": "url-banner",
  "description": "Mô tả sản phẩm",
  "stockQuantity": 50,
  "weight": 500,
  "url_images": "{\"url_images1\":\"url1\",\"url_images2\":\"url2\"}",
  "category_id": "uuid-string",
  "supplier_id": "uuid-string",
  "expire_date": "2023-12-31T00:00:00.000Z"
}
```
- **Xử lý hình ảnh**:
  - Sử dụng Cloudinary để lưu trữ hình ảnh
  - Upload hình ảnh trước khi gửi form
  - Lưu URL hình ảnh vào trường `url_images` dưới dạng JSON string
- **Validation**:
  - Tên sản phẩm: bắt buộc, tối thiểu 3 ký tự
  - Giá: bắt buộc, số dương
  - Danh mục: bắt buộc, phải tồn tại trong hệ thống
  - Nhà cung cấp: bắt buộc, phải tồn tại trong hệ thống
  - Số lượng: bắt buộc, số nguyên dương
  - Ngày hết hạn: bắt buộc, phải là ngày trong tương lai
- **Dữ liệu trả về**: Thông tin sản phẩm đã tạo
- **Giao diện**:
  - Form với các trường nhập liệu
  - Trình soạn thảo WYSIWYG cho mô tả
  - Upload hình ảnh với preview
  - Dropdown chọn danh mục và nhà cung cấp
  - Datepicker cho ngày hết hạn

### 3.2. Sửa sản phẩm

- **Mô tả**: Cập nhật thông tin sản phẩm
- **API**: `PATCH /product/:user_id`
- **Controller**: `ProductController.update()`
- **Service**: `ProductService.update()`
- **Repository**: `ProductRepository.update()`
- **Frontend**: 
  - Component: `EditProductForm.jsx`
  - Service: `editProduct(id, formData)` trong `product-service.js`
- **Quyền**: Chỉ admin mới có quyền thực hiện (`@Roles('admin')`)
- **Dữ liệu gửi lên** (`ProductUpdateDTO`): 
```json
{
  "id": "uuid-string",
  "name": "Tên sản phẩm đã cập nhật",
  "priceout": 120000,
  "banner": "url-banner-updated",
  "description": "Mô tả sản phẩm đã cập nhật",
  "stockQuantity": 60,
  "weight": 550,
  "url_images": "{\"url_images1\":\"url1-updated\",\"url_images2\":\"url2-updated\"}",
  "category_id": "uuid-string",
  "supplier_id": "uuid-string",
  "expire_date": "2024-12-31T00:00:00.000Z"
}
```
- **Xử lý**:
  - Lấy thông tin sản phẩm hiện tại trước khi cập nhật
  - Chỉ cập nhật các trường được gửi lên
  - Kiểm tra tính hợp lệ của dữ liệu
- **Validation**: Tương tự như khi thêm sản phẩm
- **Dữ liệu trả về**: Kết quả cập nhật
- **Giao diện**:
  - Form tương tự như thêm sản phẩm nhưng đã điền sẵn thông tin hiện tại
  - Hiển thị hình ảnh hiện tại với tùy chọn thay đổi

### 3.3. Xóa sản phẩm

- **Mô tả**: Xóa sản phẩm khỏi hệ thống
- **API**: `DELETE /product/:user_id/:id`
- **Controller**: `ProductController.delete()`
- **Service**: `ProductService.delete()`
- **Repository**: `ProductRepository.softDelete()` (soft delete)
- **Frontend**: 
  - Component: `ProductManagement.jsx`
  - Service: `deleteProduct(id)` trong `product-service.js`
- **Quyền**: Chỉ admin mới có quyền thực hiện (`@Roles('admin')`)
- **Tham số**: 
  - `user_id`: ID của người dùng thực hiện hành động
  - `id`: ID của sản phẩm cần xóa
- **Xử lý**:
  - Kiểm tra sản phẩm có tồn tại không
  - Kiểm tra sản phẩm có đang nằm trong đơn hàng nào không
  - Thực hiện soft delete (không xóa hoàn toàn khỏi database)
- **Dữ liệu trả về**: Kết quả xóa
- **Giao diện**:
  - Nút xóa bên cạnh mỗi sản phẩm trong danh sách
  - Hộp thoại xác nhận trước khi xóa

### 3.4. Quản lý danh sách sản phẩm

- **Mô tả**: Hiển thị danh sách sản phẩm cho quản trị viên với các chức năng thêm, sửa, xóa
- **API**: `GET /product/admin/:page/:limit`
- **Controller**: `ProductController.getAll()`
- **Service**: `ProductService.getAll()`
- **Repository**: `ProductRepository.findAndCount()`
- **Frontend**: 
  - Component: `ProductManagement.jsx`
  - Service: `getAdminProducts(page, limit, filters)` trong `product-service.js`
- **Quyền**: Chỉ admin mới có quyền truy cập
- **Tham số**:
  - `page`: Số trang hiện tại
  - `limit`: Số sản phẩm trên mỗi trang
  - `name`: Tìm kiếm theo tên (query parameter, optional)
  - `category`: Lọc theo danh mục (query parameter, optional)
  - `sort`: Trường cần sắp xếp (query parameter, optional)
  - `order`: Thứ tự sắp xếp (query parameter, optional)
- **Dữ liệu trả về**: Tương tự như API danh sách sản phẩm
- **Giao diện**:
  - Bảng hiển thị danh sách sản phẩm với các cột: ID, Tên, Giá, Danh mục, Số lượng, Ngày tạo, Hành động
  - Nút "Thêm sản phẩm" ở trên cùng
  - Thanh tìm kiếm và bộ lọc
  - Phân trang ở dưới cùng
  - Các nút "Sửa" và "Xóa" cho mỗi sản phẩm

### 3.5. Quản lý danh mục sản phẩm

- **Mô tả**: Quản lý danh mục sản phẩm (thêm, sửa, xóa)
- **API**: 
  - `GET /category/:page/:limit`: Lấy danh sách danh mục
  - `POST /category`: Thêm danh mục mới
  - `PATCH /category/:id`: Cập nhật danh mục
  - `DELETE /category/:id`: Xóa danh mục
- **Controller**: `CategoryController`
- **Service**: `CategoryService`
- **Repository**: `CategoryRepository`
- **Frontend**: 
  - Component: `CategoryManagement.jsx`
  - Service: Các hàm trong `category-service.js`
- **Quyền**: Chỉ admin mới có quyền thực hiện
- **Dữ liệu danh mục** (`CategoryEntity`):
```json
{
  "id": "uuid-string",
  "name": "Tên danh mục",
  "url_image": "url-image",
  "banner": "url-banner",
  "description": "Mô tả danh mục",
  "status": "ACTIVE"
}
```
- **Giao diện**:
  - Bảng hiển thị danh sách danh mục
  - Form thêm/sửa danh mục
  - Upload hình ảnh và banner cho danh mục

## 4. Cấu trúc dữ liệu

### 4.1. Sản phẩm (`ProductEntity`)

```typescript
@Entity({ name: 'products' })
export class ProductEntity extends BaseEntity {
  @Column({ type: 'text' })
  name: string;

  @Column({ type: 'int' })
  priceout: number;

  @Column({ type: 'text' })
  description: string;

  @Column({ type: 'int' })
  stockQuantity: number;

  @Column({ type: 'int', nullable: true })
  weight: number;

  @Column({ type: 'text', nullable: true })
  url_images: string;

  @Column({ type: 'text', nullable: true })
  banner: string;

  @Column({ type: 'date', nullable: true })
  expire_date: Date;

  @ManyToOne(() => CategoryEntity, (category) => category.products)
  @JoinColumn({ name: 'category_id' })
  category: CategoryEntity;

  @Column({ type: 'uuid' })
  category_id: string;

  @ManyToOne(() => SupplierEntity, (supplier) => supplier.products)
  @JoinColumn({ name: 'supplier_id' })
  supplier: SupplierEntity;

  @Column({ type: 'uuid' })
  supplier_id: string;

  @OneToMany(() => ImportProductEntity, (importProduct) => importProduct.product)
  importProducts: ImportProductEntity[];

  @OneToMany(() => OrderProductEntity, (orderProduct) => orderProduct.product)
  orderProducts: OrderProductEntity[];

  @OneToMany(() => CartEntity, (cart) => cart.product)
  carts: CartEntity[];
}
```

### 4.2. Danh mục (`CategoryEntity`)

```typescript
@Entity({ name: 'categories' })
export class CategoryEntity extends BaseEntity {
  @Column({ type: 'text' })
  name: string;

  @Column({ type: 'text', nullable: true })
  url_image: string;

  @Column({ type: 'text', nullable: true })
  banner: string;

  @Column({ type: 'text', nullable: true })
  description: string;

  @Column({ type: 'enum', enum: Status, default: Status.ACTIVE })
  status: Status;

  @OneToMany(() => ProductEntity, (product) => product.category)
  products: ProductEntity[];
}
```

### 4.3. Nhà cung cấp (`SupplierEntity`)

```typescript
@Entity({ name: 'suppliers' })
export class SupplierEntity extends BaseEntity {
  @Column({ type: 'text' })
  name: string;

  @Column({ type: 'text' })
  address: string;

  @Column({ type: 'text' })
  phone: string;

  @Column({ type: 'text', nullable: true })
  email: string;

  @Column({ type: 'enum', enum: Status, default: Status.ACTIVE })
  status: Status;

  @OneToMany(() => ProductEntity, (product) => product.supplier)
  products: ProductEntity[];

  @OneToMany(() => ImportEntity, (import_) => import_.supplier)
  imports: ImportEntity[];
}
```

## 5. Xử lý hình ảnh sản phẩm

### 5.1. Lưu trữ hình ảnh

- **Công nghệ sử dụng**: Cloudinary
- **Định dạng lưu trữ**: URL của hình ảnh được lưu trong database
- **Cấu trúc dữ liệu**: Trường `url_images` lưu dưới dạng JSON string
```json
{
  "url_images1": "https://res.cloudinary.com/example/image/upload/v1234567890/product1.jpg",
  "url_images2": "https://res.cloudinary.com/example/image/upload/v1234567890/product2.jpg"
}
```

### 5.2. Upload hình ảnh

- **Frontend**:
  - Sử dụng component `ImageUploader.jsx`
  - Hỗ trợ kéo thả (drag & drop) và chọn file
  - Preview hình ảnh trước khi upload
  - Giới hạn kích thước file: tối đa 5MB
  - Định dạng hỗ trợ: JPG, PNG, WEBP
- **Backend**:
  - Sử dụng Cloudinary API để upload hình ảnh
  - Tạo signed upload URL để upload trực tiếp từ client
  - Xử lý resize và optimize hình ảnh tự động

### 5.3. Hiển thị hình ảnh

- **Xử lý URL hình ảnh**:
```javascript
// Chuyển đổi từ JSON string sang object
const parseUrlImages = (urlImagesString) => {
  if (!urlImagesString) return { url_image1: "", url_image2: "" };
  
  try {
    const cleanedUrlImages = urlImagesString.replace(/\\\"/g, '"');
    const urlImages = JSON.parse(cleanedUrlImages);
    return {
      url_image1: urlImages.url_images1 || "",
      url_image2: urlImages.url_images2 || ""
    };
  } catch (error) {
    console.error("Error parsing url_images:", error);
    return { url_image1: "", url_image2: "" };
  }
};
```

- **Responsive Images**:
  - Sử dụng Cloudinary transformation URLs để tải hình ảnh với kích thước phù hợp
  - Lazy loading cho hình ảnh để tối ưu hiệu suất
  - Placeholder cho hình ảnh đang tải

## 6. Tối ưu hiệu suất

### 6.1. Caching

- **Frontend**:
  - Sử dụng React Query để cache dữ liệu API
  - Local storage để lưu trữ dữ liệu không thay đổi thường xuyên (danh mục, cấu hình)
- **Backend**:
  - Redis cache cho các truy vấn phổ biến
  - Cache danh sách sản phẩm và danh mục

### 6.2. Phân trang và lazy loading

- **Phân trang**:
  - Mặc định: 10 sản phẩm mỗi trang
  - Người dùng có thể thay đổi số lượng hiển thị (10, 20, 50)
  - Sử dụng cursor-based pagination cho hiệu suất tốt hơn
- **Lazy loading**:
  - Tải hình ảnh khi cần thiết
  - Infinite scroll cho danh sách sản phẩm trên mobile

### 6.3. Indexing database

- **Indexes**:
  - Index cho `name` để tối ưu tìm kiếm
  - Index cho `category_id` để tối ưu lọc
  - Index cho `createdAt` để tối ưu sắp xếp

## 7. Xử lý lỗi và validation

### 7.1. Validation dữ liệu

- **Backend**:
  - Sử dụng class-validator để validate DTO
  - Custom validators cho các trường đặc biệt
- **Frontend**:
  - Formik và Yup để validate form
  - Hiển thị thông báo lỗi ngay dưới trường nhập liệu

### 7.2. Xử lý lỗi

- **Backend**:
  - Global exception filter để xử lý lỗi
  - Response format chuẩn cho lỗi
```json
{
  "success": false,
  "status": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "name",
      "message": "Name must be at least 3 characters"
    }
  ]
}
```
- **Frontend**:
  - Interceptor để xử lý lỗi từ API
  - Toast notifications để hiển thị lỗi
  - Retry mechanism cho các lỗi mạng

## 8. Bảo mật

### 8.1. Phân quyền

- **Roles**:
  - `admin`: Toàn quyền quản lý sản phẩm
  - `user`: Chỉ có quyền xem sản phẩm
- **Guards**:
  - `JwtAuthGuard`: Xác thực người dùng
  - `RolesGuard`: Kiểm tra quyền

### 8.2. Validation input

- Sanitize input để tránh XSS
- Rate limiting để tránh brute force
- CSRF protection
