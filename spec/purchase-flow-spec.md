# Đặc tả kỹ thuật: Module Mua Hàng

## 1. Tổng quan

Module Mua Hàng là thành phần quan trọng trong hệ thống thương mại điện tử, cho phép người dùng:
- Thêm sản phẩm vào giỏ hàng
- Quản lý giỏ hàng (thêm, sửa, xóa)
- Thanh toán đơn hàng
- Theo dõi trạng thái đơn hàng

### 1.1. Mục tiêu
- Cung cấp quy trình mua hàng đơn giản, dễ sử dụng
- Hỗ trợ nhiều phương thức thanh toán
- Đảm bảo tính bảo mật và chính xác trong quá trình thanh toán
- Cung cấp thông tin đơn hàng rõ ràng cho người dùng

### 1.2. Công nghệ sử dụng
- **Backend**: NestJS, TypeORM, MySQL
- **Frontend**: ReactJS, TailwindCSS
- **API**: RESTful API
- **Xác thực**: JWT
- **Thanh toán**: MOMO, Thanh toán khi nhận hàng (COD)

## 2. Quản lý giỏ hàng

### 2.1. Thêm sản phẩm vào giỏ hàng

#### 2.1.1. Thêm từ trang danh sách sản phẩm
- **Mô tả**: Thêm sản phẩm vào giỏ hàng từ trang danh sách sản phẩm
- **API**: `POST /cart/add-to-cart/:user_id`
- **Controller**: `CartController.addToCart()`
- **Service**: `CartService.create()`
- **Repository**: `CartRepository.save()`
- **Quyền**: Người dùng đã đăng nhập (`@Roles('user')`)
- **Dữ liệu gửi lên** (`CreateCartDto`):
  - `product_id`: ID sản phẩm
  - `quantity`: Số lượng (mặc định: 1)
  - `user_id`: ID người dùng
- **Xử lý**:
  - Kiểm tra sản phẩm đã tồn tại trong giỏ hàng chưa
  - Nếu đã tồn tại, cập nhật số lượng
  - Nếu chưa tồn tại, thêm mới vào giỏ hàng
- **Dữ liệu trả về**: Thông tin sản phẩm đã thêm vào giỏ hàng

#### 2.1.2. Thêm từ trang chi tiết sản phẩm
- **Mô tả**: Thêm sản phẩm vào giỏ hàng từ trang chi tiết sản phẩm
- **API**: `POST /cart/add-to-cart/:user_id`
- **Controller**: `CartController.addToCart()`
- **Service**: `CartService.create()`
- **Repository**: `CartRepository.save()`
- **Quyền**: Người dùng đã đăng nhập (`@Roles('user')`)
- **Dữ liệu gửi lên** (`CreateCartDto`):
  - `product_id`: ID sản phẩm
  - `quantity`: Số lượng (có thể chọn số lượng > 1)
  - `user_id`: ID người dùng
- **Xử lý**: Tương tự như thêm từ trang danh sách sản phẩm
- **Dữ liệu trả về**: Thông tin sản phẩm đã thêm vào giỏ hàng

### 2.2. Hiển thị giỏ hàng

- **Mô tả**: Hiển thị danh sách sản phẩm trong giỏ hàng của người dùng
- **API**: `GET /cart/all-product/:user_id`
- **Controller**: `CartController.getAllProductInCart()`
- **Service**: `CartService.getListProduct()`
- **Repository**: `CartRepository.findAndCount()`
- **Quyền**: Người dùng đã đăng nhập (`@Roles('user')`)
- **Tham số**: `user_id`: ID người dùng
- **Dữ liệu trả về**:
  - Danh sách sản phẩm trong giỏ hàng
  - Tổng số sản phẩm trong giỏ hàng

### 2.3. Cập nhật số lượng sản phẩm

- **Mô tả**: Cập nhật số lượng sản phẩm trong giỏ hàng
- **API**: `PATCH /cart/:user_id`
- **Controller**: `CartController.update()`
- **Service**: `CartService.update()`
- **Repository**: `CartRepository.update()`
- **Quyền**: Người dùng đã đăng nhập (`@Roles('user')`)
- **Dữ liệu gửi lên** (`UpdateCartDto`):
  - `id`: ID của item trong giỏ hàng
  - `quantity`: Số lượng mới
- **Xử lý**:
  - Kiểm tra item có tồn tại trong giỏ hàng không
  - Cập nhật số lượng mới
- **Dữ liệu trả về**: Thông tin item đã cập nhật

### 2.4. Xóa sản phẩm khỏi giỏ hàng

- **Mô tả**: Xóa một hoặc nhiều sản phẩm khỏi giỏ hàng
- **API**: `DELETE /cart/:user_id`
- **Controller**: `CartController.delete()`
- **Service**: `CartService.deleteProductsInCart()`
- **Repository**: `CartRepository.delete()`
- **Quyền**: Người dùng đã đăng nhập (`@Roles('user')`)
- **Dữ liệu gửi lên** (`DeleteCartDto`):
  - `cart_ids`: Mảng ID của các item cần xóa
- **Xử lý**:
  - Kiểm tra các item có tồn tại trong giỏ hàng không
  - Xóa các item khỏi giỏ hàng
- **Dữ liệu trả về**: Kết quả xóa

## 3. Quy trình thanh toán

### 3.1. Trang thanh toán (Checkout)

- **Mô tả**: Hiển thị trang thanh toán với thông tin đơn hàng và các phương thức thanh toán
- **Component**: `Checkout.jsx`
- **Dữ liệu hiển thị**:
  - Danh sách sản phẩm đã chọn
  - Tổng tiền
  - Địa chỉ giao hàng
  - Phương thức thanh toán

### 3.2. Tạo đơn hàng

#### 3.2.1. Thanh toán khi nhận hàng (COD)
- **Mô tả**: Tạo đơn hàng với phương thức thanh toán COD
- **API**: `POST /order/:user_id`
- **Controller**: `OrderController.createOrder()`
- **Service**: `OrderService.createOrder()`
- **Repository**: `OrderRepository.save()`
- **Quyền**: Người dùng đã đăng nhập (`@Roles('user')`)
- **Dữ liệu gửi lên** (`CreateOrderDto`):
  - `totalPrice`: Tổng tiền đơn hàng
  - `paymentMethod`: Phương thức thanh toán (COD)
  - `user_id`: ID người dùng
  - `location_id`: ID địa chỉ giao hàng
  - `orderStatus`: Trạng thái đơn hàng (Checking)
  - `paymentStatus`: Trạng thái thanh toán (Unpaid)
  - `products`: Mảng sản phẩm trong đơn hàng
    - `product_id`: ID sản phẩm
    - `quantity`: Số lượng
    - `priceout`: Giá bán
- **Xử lý**:
  - Tạo đơn hàng mới
  - Tạo các chi tiết đơn hàng
  - Xóa sản phẩm khỏi giỏ hàng
  - Tạo thông báo đơn hàng mới
- **Dữ liệu trả về**: Thông tin đơn hàng đã tạo

#### 3.2.2. Thanh toán qua MOMO
- **Mô tả**: Tạo đơn hàng và thanh toán qua MOMO
- **API**: `POST /momo/create-payment`
- **Controller**: `MomoController.createPayment()`
- **Service**: `MomoService.createPayment()`
- **Quyền**: Người dùng đã đăng nhập
- **Dữ liệu gửi lên**:
  - `order`: Thông tin đơn hàng (tương tự như COD)
  - `amount`: Số tiền thanh toán
  - `redirectUrl`: URL chuyển hướng sau khi thanh toán
  - `ipnUrl`: URL callback từ MOMO
- **Xử lý**:
  - Tạo đơn hàng mới
  - Gọi API của MOMO để tạo thanh toán
  - Xóa sản phẩm khỏi giỏ hàng
  - Chuyển hướng người dùng đến trang thanh toán của MOMO
- **Dữ liệu trả về**:
  - Thông tin đơn hàng đã tạo
  - Thông tin thanh toán từ MOMO (shortLink, message)

### 3.3. Xử lý callback từ MOMO

- **Mô tả**: Xử lý callback từ MOMO sau khi thanh toán
- **API**: `POST /momo/callback-payment`
- **Controller**: `MomoController.handleCallback()`
- **Service**: `MomoService.handleCallback()`
- **Xử lý**:
  - Kiểm tra tính hợp lệ của callback
  - Cập nhật trạng thái đơn hàng
  - Cập nhật trạng thái thanh toán
- **Dữ liệu trả về**: Kết quả xử lý callback

### 3.4. Trang thông báo đặt hàng thành công

- **Mô tả**: Hiển thị trang thông báo đặt hàng thành công
- **Component**: `OrderSuccess.jsx`
- **Dữ liệu hiển thị**:
  - Mã đơn hàng
  - Tổng tiền
  - Phương thức thanh toán
  - Trạng thái thanh toán
  - Nút để xem chi tiết đơn hàng

## 4. Quản lý đơn hàng

### 4.1. Danh sách đơn hàng của người dùng

- **Mô tả**: Hiển thị danh sách đơn hàng của người dùng
- **API**: `POST /order/all-user-order/:user_id`
- **Controller**: `OrderController.getAllOrder()`
- **Service**: `OrderService.getAllOrder()`
- **Repository**: `OrderRepository.find()`
- **Quyền**: Người dùng đã đăng nhập (`@Roles('user')`)
- **Tham số**: `user_id`: ID người dùng
- **Dữ liệu gửi lên** (`OrderAllOrderDto`):
  - `page`: Số trang
  - `limit`: Số đơn hàng trên mỗi trang
  - `orderStatus`: Trạng thái đơn hàng (optional)
- **Dữ liệu trả về**:
  - Danh sách đơn hàng
  - Tổng số đơn hàng
  - Thống kê theo trạng thái đơn hàng

### 4.2. Chi tiết đơn hàng

- **Mô tả**: Hiển thị chi tiết đơn hàng
- **API**: `GET /order/detail/:id`
- **Controller**: `OrderController.getOrderDetail()`
- **Service**: `OrderService.getOrderDetail()`
- **Repository**: `OrderRepository.findOne()`
- **Quyền**: Người dùng đã đăng nhập (`@Roles('user')`)
- **Tham số**: `id`: ID đơn hàng
- **Dữ liệu trả về**:
  - Thông tin đơn hàng
  - Danh sách sản phẩm trong đơn hàng
  - Thông tin địa chỉ giao hàng
  - Thông tin thanh toán

### 4.3. Cập nhật đơn hàng

- **Mô tả**: Cập nhật trạng thái đơn hàng
- **API**: `PATCH /order/:user_id`
- **Controller**: `OrderController.updateOrder()`
- **Service**: `OrderService.updateOrder()`
- **Repository**: `OrderRepository.update()`
- **Quyền**: Người dùng, admin, nhân viên (`@Roles('user', 'admin', 'employee')`)
- **Dữ liệu gửi lên** (`UpdateOrderDTO`):
  - `id`: ID đơn hàng
  - `orderStatus`: Trạng thái đơn hàng mới
  - `paymentStatus`: Trạng thái thanh toán mới (optional)
- **Xử lý**:
  - Kiểm tra đơn hàng có tồn tại không
  - Cập nhật trạng thái đơn hàng
  - Cập nhật trạng thái thanh toán (nếu có)
- **Dữ liệu trả về**: Thông tin đơn hàng đã cập nhật

## 5. Cấu trúc dữ liệu

### 5.1. Giỏ hàng (`CartEntity`)

```typescript
@Entity({ name: 'cart_product' })
export class CartEntity extends BaseEntity {
  @Column({ type: 'int' })
  quantity: number;

  @ManyToOne(() => ProductEntity, (product) => product.carts)
  @JoinColumn({ name: 'product_id' })
  product: ProductEntity;

  @Column({ type: 'uuid' })
  product_id: string;

  @ManyToOne(() => UserEntity, (user) => user.carts)
  @JoinColumn({ name: 'user_id' })
  user: UserEntity;

  @Column({ type: 'uuid' })
  user_id: string;
}
```

### 5.2. Đơn hàng (`OrderEntity`)

```typescript
@Entity({ name: 'orders' })
export class OrderEntity extends BaseEntity {
  @Column({ type: 'text' })
  order_code: string;

  @Column({ type: 'int' })
  total_price: number;

  @Column({ type: 'enum', enum: OrderStatus, default: OrderStatus.Checking })
  orderStatus: OrderStatus;

  @Column({ type: 'enum', enum: PaymentMethod, default: PaymentMethod.CashOnDelivery })
  payment_method: PaymentMethod;

  @Column({ type: 'enum', enum: PaymentStatus, default: PaymentStatus.Unpaid })
  paymentStatus: PaymentStatus;

  @ManyToOne(() => UserEntity, (user) => user.orders)
  @JoinColumn({ name: 'user_id' })
  user: UserEntity;

  @Column({ type: 'uuid' })
  user_id: string;

  @ManyToOne(() => UserEntity, (employee) => employee.employeeOrders)
  @JoinColumn({ name: 'employee_id' })
  employee: UserEntity;

  @Column({ type: 'uuid', nullable: true })
  employee_id: string;

  @ManyToOne(() => LocationEntity, (location) => location.orders)
  @JoinColumn({ name: 'location_id' })
  location: LocationEntity;

  @Column({ type: 'uuid' })
  location_id: string;

  @OneToMany(() => OrderProductEntity, (orderProduct) => orderProduct.order)
  orderProducts: OrderProductEntity[];
}
```

### 5.3. Chi tiết đơn hàng (`OrderProductEntity`)

```typescript
@Entity({ name: 'order_product' })
export class OrderProductEntity extends BaseEntity {
  @Column({ type: 'int' })
  quantity: number;

  @Column({ type: 'int' })
  priceout: number;

  @ManyToOne(() => ProductEntity, (product) => product.orderProducts)
  @JoinColumn({ name: 'product_id' })
  product: ProductEntity;

  @Column({ type: 'uuid' })
  product_id: string;

  @ManyToOne(() => OrderEntity, (order) => order.orderProducts)
  @JoinColumn({ name: 'order_id' })
  order: OrderEntity;

  @Column({ type: 'uuid' })
  order_id: string;
}
```

### 5.4. Enum và Constants

#### 5.4.1. Trạng thái đơn hàng (`OrderStatus`)

```typescript
export enum OrderStatus {
  All = 'All',
  Checking = 'Checking',
  InTransit = 'InTransit',
  Delivered = 'Delivered',
  Canceled = 'Canceled',
}
```

#### 5.4.2. Phương thức thanh toán (`PaymentMethod`)

```typescript
export enum PaymentMethod {
  CashOnDelivery = 'Thanh toán khi nhận hàng',
  BankTransfer = 'Thanh toán qua MOMO',
}
```

#### 5.4.3. Trạng thái thanh toán (`PaymentStatus`)

```typescript
export enum PaymentStatus {
  Paid = 'Paid',
  Unpaid = 'Unpaid',
}
```

## 6. Luồng xử lý thanh toán

### 6.1. Thanh toán khi nhận hàng (COD)

1. Người dùng chọn sản phẩm và thêm vào giỏ hàng
2. Người dùng vào trang giỏ hàng, chọn sản phẩm cần thanh toán
3. Người dùng chuyển đến trang thanh toán (Checkout)
4. Người dùng chọn phương thức thanh toán "Thanh toán khi nhận hàng"
5. Người dùng nhấn nút "Đặt hàng"
6. Hệ thống tạo đơn hàng mới với trạng thái "Checking" và trạng thái thanh toán "Unpaid"
7. Hệ thống xóa sản phẩm đã đặt khỏi giỏ hàng
8. Hệ thống chuyển hướng người dùng đến trang thông báo đặt hàng thành công
9. Người dùng có thể xem chi tiết đơn hàng trong trang quản lý đơn hàng

### 6.2. Thanh toán qua MOMO

1. Người dùng chọn sản phẩm và thêm vào giỏ hàng
2. Người dùng vào trang giỏ hàng, chọn sản phẩm cần thanh toán
3. Người dùng chuyển đến trang thanh toán (Checkout)
4. Người dùng chọn phương thức thanh toán "Thanh toán qua MOMO"
5. Người dùng nhấn nút "Đặt hàng"
6. Hệ thống tạo đơn hàng mới với trạng thái "Checking" và trạng thái thanh toán "Unpaid"
7. Hệ thống gọi API của MOMO để tạo thanh toán
8. Hệ thống xóa sản phẩm đã đặt khỏi giỏ hàng
9. Hệ thống chuyển hướng người dùng đến trang thanh toán của MOMO
10. Người dùng hoàn thành thanh toán trên trang MOMO
11. MOMO gửi callback đến hệ thống
12. Hệ thống cập nhật trạng thái thanh toán của đơn hàng thành "Paid"
13. Người dùng được chuyển hướng về trang thông báo đặt hàng thành công
14. Người dùng có thể xem chi tiết đơn hàng trong trang quản lý đơn hàng

## 7. Bảo mật

### 7.1. Phân quyền

- **Roles**:
  - `user`: Có quyền thêm sản phẩm vào giỏ hàng, thanh toán, xem đơn hàng của mình
  - `admin`: Có quyền xem và quản lý tất cả đơn hàng
  - `employee`: Có quyền xem và cập nhật trạng thái đơn hàng
- **Guards**:
  - `JwtAuthGuard`: Xác thực người dùng
  - `RolesGuard`: Kiểm tra quyền

### 7.2. Xử lý thanh toán an toàn

- **MOMO Payment**:
  - Sử dụng HMAC SHA256 để tạo chữ ký
  - Xác thực callback từ MOMO
  - Lưu trữ thông tin thanh toán an toàn

## 8. Xử lý lỗi

- **Global exception filter** để xử lý lỗi
- **Response format chuẩn** cho lỗi:
```json
{
  "success": false,
  "status": 400,
  "message": "Failed to create payment: Invalid amount",
  "errors": []
}
```
