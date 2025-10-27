# Amazon FSx for Windows File Server Tutorial

Hướng dẫn triển khai môi trường tutorial Amazon FSx for Windows File Server sử dụng AWS CloudFormation.

## Mô tả

Template CloudFormation này tạo ra một môi trường hoàn chỉnh để học và thực hành với Amazon FSx for Windows File Server, bao gồm:

- VPC với public và private subnets
- AWS Managed Microsoft Active Directory
- Amazon FSx for Windows File Server
- EC2 Windows instance với các công cụ benchmark
- Security groups và IAM roles
- CloudWatch dashboard để monitoring

## Yêu cầu trước khi triển khai

- AWS Account với quyền tạo các resources: VPC, EC2, FSx, Directory Service, IAM, Lambda
- Chọn AWS Region hỗ trợ Amazon FSx for Windows File Server
- Đảm bảo service limits đủ cho các resources sẽ tạo

## Cách triển khai

### 1. Tải template lên CloudFormation

#### Sử dụng AWS Console:
1. Đăng nhập vào AWS Console
2. Chuyển đến CloudFormation service
3. Click **Create stack** → **With new resources (standard)**
4. Chọn **Upload a template file**
5. Click **Choose file** và chọn `fsxw-tutorial.yaml`
6. Click **Next**

#### Sử dụng AWS CLI:
```bash
aws cloudformation create-stack \
  --stack-name fsxw-tutorial \
  --template-body file://fsxw-tutorial.yaml \
  --parameters ParameterKey=AvailabilityZones,ParameterValue="us-east-1a,us-east-1b" \
               ParameterKey=InstanceType,ParameterValue="m5.xlarge" \
               ParameterKey=VpcCidr,ParameterValue="10.0.0.0/16" \
  --capabilities CAPABILITY_IAM
```

### 2. Cấu hình Parameters

Khi tạo stack, bạn cần cung cấp các parameters sau:

| Parameter | Mô tả | Giá trị mặc định |
|-----------|-------|------------------|
| **AvailabilityZones** | Chọn 2 AZ để tạo subnets | *Bắt buộc chọn* |
| **InstanceType** | Loại EC2 instance | m5.xlarge |
| **VpcCidr** | CIDR block cho VPC | 10.0.0.0/16 |

### 3. Xem lại và tạo stack

1. Xem lại các thông tin cấu hình
2. Tích chọn **I acknowledge that AWS CloudFormation might create IAM resources**
3. Click **Create stack**

## Thời gian triển khai

- **Tổng thời gian**: 45-60 phút
- **Active Directory**: 20-30 phút
- **FSx File System**: 10-15 phút  
- **EC2 Domain Join**: 5-10 phút

## Sau khi triển khai thành công

### 1. Truy cập Windows Instance
```bash
# Lấy thông tin instance
aws ec2 describe-instances --filters "Name=tag:Name,Values=Windows Instance 0"

# Lấy password để RDP
aws ec2 get-password-data --instance-id <instance-id> --priv-launch-key <key-pair>
```

### 2. Thông tin Active Directory
- **Domain**: example.com
- **Admin username**: admin@example.com  
- **Password**: Được tạo tự động trong AWS Secrets Manager

### 3. Truy cập FSx File System
Từ Windows instance, mount FSx file system:
```powershell
# Tìm DNS name của FSx
$fsxDnsName = "fs-xxxxxxxxx.example.com"

# Mount drive
net use Z: \\$fsxDnsName\share
```

### 4. CloudWatch Dashboard
- Tên dashboard: `<region>-<filesystem-id>`
- Monitoring: throughput, IOPS, storage capacity

## Công cụ benchmark có sẵn

Trên Windows instance đã cài đặt sẵn:
- **DiskSpd**: `C:\Tools\DiskSpd-2.0.21a\`
- **FIO**: `C:\Tools\fio-3.16-x64\`

## Dọn dẹp resources

### Sử dụng Console:
1. Vào CloudFormation console
2. Chọn stack `fsxw-tutorial`
3. Click **Delete**

### Sử dụng CLI:
```bash
aws cloudformation delete-stack --stack-name fsxw-tutorial
```

**⚠️ Lưu ý**: Việc xóa stack sẽ xóa tất cả data trong FSx file system và không thể khôi phục.

## Troubleshooting

### Stack tạo thất bại
1. Kiểm tra CloudFormation Events tab để xem lỗi chi tiết
2. Đảm bảo region được chọn hỗ trợ tất cả services
3. Kiểm tra service limits và quotas

### Không thể domain join
1. Kiểm tra DHCP options đã được associate với VPC
2. Verify DNS settings trên instance
3. Kiểm tra security group rules

### FSx mount thất bại
1. Verify instance đã join domain thành công
2. Kiểm tra security group cho port 445 (SMB)
3. Đảm bảo FSx và instance cùng VPC

## Chi phí ước tính

Với cấu hình mặc định (us-east-1):
- **FSx (1TB, 64 MB/s)**: ~$300/tháng
- **EC2 m5.xlarge**: ~$140/tháng  
- **Managed AD**: ~$110/tháng
- **VPC, EBS**: ~$20/tháng

**Tổng**: ~$570/tháng

## Liên hệ

Tác giả: Darryl Osborne (darrylo@amazon.com)