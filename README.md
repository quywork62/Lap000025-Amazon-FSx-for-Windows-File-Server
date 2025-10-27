# Amazon FSx for Windows File Server Tutorial

---

## 🇻🇳 PHẦN 1: TIẾNG VIỆT

### Mô tả

Template CloudFormation này tạo ra một môi trường hoàn chỉnh để học và thực hành với Amazon FSx for Windows File Server, bao gồm:

- VPC với public và private subnets
- AWS Managed Microsoft Active Directory
- Amazon FSx for Windows File Server
- EC2 Windows instance với các công cụ benchmark
- Security groups và IAM roles
- CloudWatch dashboard để monitoring

### Yêu cầu trước khi triển khai

- AWS Account với quyền tạo các resources: VPC, EC2, FSx, Directory Service, IAM, Lambda
- Chọn AWS Region hỗ trợ Amazon FSx for Windows File Server
- Đảm bảo service limits đủ cho các resources sẽ tạo

### Cách triển khai

#### 1. Tải template lên CloudFormation

**Sử dụng AWS Console:**
1. Đăng nhập vào AWS Console
2. Chuyển đến CloudFormation service
3. Click **Create stack** → **With new resources (standard)**
4. Chọn **Upload a template file**
5. Click **Choose file** và chọn `fsxw-tutorial.yaml`
6. Click **Next**

**Sử dụng AWS CLI:**
```bash
aws cloudformation create-stack \
  --stack-name fsxw-tutorial \
  --template-body file://fsxw-tutorial.yaml \
  --parameters ParameterKey=AvailabilityZones,ParameterValue="us-east-1a,us-east-1b" \
               ParameterKey=InstanceType,ParameterValue="m5.xlarge" \
               ParameterKey=VpcCidr,ParameterValue="10.0.0.0/16" \
  --capabilities CAPABILITY_IAM
```

#### 2. Cấu hình Parameters

| Parameter | Mô tả | Giá trị mặc định |
|-----------|-------|------------------|
| **AvailabilityZones** | Chọn 2 AZ để tạo subnets | *Bắt buộc chọn* |
| **InstanceType** | Loại EC2 instance | m5.xlarge |
| **VpcCidr** | CIDR block cho VPC | 10.0.0.0/16 |

#### 3. Xem lại và tạo stack

1. Xem lại các thông tin cấu hình
2. Tích chọn **I acknowledge that AWS CloudFormation might create IAM resources**
3. Click **Create stack**

### Thời gian triển khai

- **Tổng thời gian**: 45-60 phút
- **Active Directory**: 20-30 phút
- **FSx File System**: 10-15 phút  
- **EC2 Domain Join**: 5-10 phút

### Sau khi triển khai thành công

#### 1. Truy cập Windows Instance
```bash
# Lấy thông tin instance
aws ec2 describe-instances --filters "Name=tag:Name,Values=Windows Instance 0"

# Lấy password để RDP
aws ec2 get-password-data --instance-id <instance-id> --priv-launch-key <key-pair>
```

#### 2. Thông tin Active Directory
- **Domain**: example.com
- **Admin username**: admin@example.com  
- **Password**: Được tạo tự động trong AWS Secrets Manager

#### 3. Truy cập FSx File System
```powershell
# Tìm DNS name của FSx
$fsxDnsName = "fs-xxxxxxxxx.example.com"

# Mount drive
net use Z: \\$fsxDnsName\share
```

### Công cụ benchmark có sẵn

- **DiskSpd**: `C:\Tools\DiskSpd-2.0.21a\`
- **FIO**: `C:\Tools\fio-3.16-x64\`

### Dọn dẹp resources

```bash
aws cloudformation delete-stack --stack-name fsxw-tutorial
```

**⚠️ Lưu ý**: Việc xóa stack sẽ xóa tất cả data trong FSx file system và không thể khôi phục.

### Chi phí ước tính (us-east-1)

- **FSx (1TB, 64 MB/s)**: ~$300/tháng
- **EC2 m5.xlarge**: ~$140/tháng  
- **Managed AD**: ~$110/tháng
- **VPC, EBS**: ~$20/tháng

**Tổng**: ~$570/tháng

---

## 🇺🇸 PART 2: ENGLISH

### Description

This CloudFormation template creates a complete environment for learning and practicing with Amazon FSx for Windows File Server, including:

- VPC with public and private subnets
- AWS Managed Microsoft Active Directory
- Amazon FSx for Windows File Server
- EC2 Windows instance with benchmark tools
- Security groups and IAM roles
- CloudWatch dashboard for monitoring

### Prerequisites

- AWS Account with permissions to create: VPC, EC2, FSx, Directory Service, IAM, Lambda resources
- Select AWS Region that supports Amazon FSx for Windows File Server
- Ensure sufficient service limits for resources to be created

### Deployment Steps

#### 1. Upload template to CloudFormation

**Using AWS Console:**
1. Log in to AWS Console
2. Navigate to CloudFormation service
3. Click **Create stack** → **With new resources (standard)**
4. Select **Upload a template file**
5. Click **Choose file** and select `fsxw-tutorial.yaml`
6. Click **Next**

**Using AWS CLI:**
```bash
aws cloudformation create-stack \
  --stack-name fsxw-tutorial \
  --template-body file://fsxw-tutorial.yaml \
  --parameters ParameterKey=AvailabilityZones,ParameterValue="us-east-1a,us-east-1b" \
               ParameterKey=InstanceType,ParameterValue="m5.xlarge" \
               ParameterKey=VpcCidr,ParameterValue="10.0.0.0/16" \
  --capabilities CAPABILITY_IAM
```

#### 2. Configure Parameters

| Parameter | Description | Default Value |
|-----------|-------------|---------------|
| **AvailabilityZones** | Select 2 AZs to create subnets | *Required selection* |
| **InstanceType** | EC2 instance type | m5.xlarge |
| **VpcCidr** | CIDR block for VPC | 10.0.0.0/16 |

#### 3. Review and create stack

1. Review configuration information
2. Check **I acknowledge that AWS CloudFormation might create IAM resources**
3. Click **Create stack**

### Deployment Time

- **Total time**: 45-60 minutes
- **Active Directory**: 20-30 minutes
- **FSx File System**: 10-15 minutes  
- **EC2 Domain Join**: 5-10 minutes

### After Successful Deployment

#### 1. Access Windows Instance
```bash
# Get instance information
aws ec2 describe-instances --filters "Name=tag:Name,Values=Windows Instance 0"

# Get password for RDP
aws ec2 get-password-data --instance-id <instance-id> --priv-launch-key <key-pair>
```

#### 2. Active Directory Information
- **Domain**: example.com
- **Admin username**: admin@example.com  
- **Password**: Auto-generated in AWS Secrets Manager

#### 3. Access FSx File System
```powershell
# Find FSx DNS name
$fsxDnsName = "fs-xxxxxxxxx.example.com"

# Mount drive
net use Z: \\$fsxDnsName\share
```

#### 4. CloudWatch Dashboard
- Dashboard name: `<region>-<filesystem-id>`
- Monitoring: throughput, IOPS, storage capacity

### Available Benchmark Tools

Pre-installed on Windows instance:
- **DiskSpd**: `C:\Tools\DiskSpd-2.0.21a\`
- **FIO**: `C:\Tools\fio-3.16-x64\`

### Resource Cleanup

**Using Console:**
1. Go to CloudFormation console
2. Select `fsxw-tutorial` stack
3. Click **Delete**

**Using CLI:**
```bash
aws cloudformation delete-stack --stack-name fsxw-tutorial
```

**⚠️ Warning**: Deleting the stack will remove all data in FSx file system and cannot be recovered.

### Troubleshooting

#### Stack creation failed
1. Check CloudFormation Events tab for detailed errors
2. Ensure selected region supports all services
3. Check service limits and quotas

#### Cannot domain join
1. Verify DHCP options are associated with VPC
2. Verify DNS settings on instance
3. Check security group rules

#### FSx mount failed
1. Verify instance successfully joined domain
2. Check security group for port 445 (SMB)
3. Ensure FSx and instance are in same VPC

### Cost Estimation

With default configuration (us-east-1):
- **FSx (1TB, 64 MB/s)**: ~$300/month
- **EC2 m5.xlarge**: ~$140/month  
- **Managed AD**: ~$110/month
- **VPC, EBS**: ~$20/month

**Total**: ~$570/month

### Contact

Author: Darryl Osborne (darrylo@amazon.com)