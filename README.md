<!-- note marker start -->
**NOTE**: This repo contains only the documentation for the private BoltsOps Pro repo code.
Original file: https://github.com/boltopspro/efs/blob/master/README.md
The docs are publish so they are available for interested customers.
For access to the source code, you must be a paying BoltOps Pro subscriber.
If are interested, you can contact us at contact@boltops.com or https://www.boltops.com

<!-- note marker end -->

# EFS FileSystem CloudFormation Blueprint

[![BoltOps Badge](https://img.boltops.com/boltops/badges/boltops-badge.png)](https://www.boltops.com)

![CodeBuild](https://codebuild.us-west-2.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoiUEpvRTcwOWlWRjU4dXlPYXVvVDFERXpZaG5tK2Rlc2U4UFp5VXVWYzVHZnc3Nmt5WVlIRmRtTm5DdW92M3NUbDgyVEpzeUZWY0dzaDdoK3JQbGJFQks0PSIsIml2UGFyYW1ldGVyU3BlYyI6InJnejNBVHNlRy9adW5KQU0iLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master)

This blueprint provisions an [EFS FileSystem](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-efs-filesystem.html):

* Several [EFS FileSystem](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-efs-filesystem.html)  properties are configurable with [Parameters](https://lono.cloud/docs/configs/params/). Additionally, properties that require further customization are configurable with [Variables](https://lono.cloud/docs/configs/shared-variables/).  The blueprint is configurable for your needs.
* It creates a configurable mount of [MountTargets](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-efs-mounttarget.html) also. This is configurable with `@subnets`.
* A managed [Security Group](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html) is created if no existing SecurityGroup ids are provided.
* The FileSystem is encrypted by default. You can configure your own `KmsKeyId` to use.
* For safety reasons, the DeletionPolicy is Retain for the FileSystem.  This can be adjusted with `@delete_policy = "Delete"`

## Usage

1. Add blueprint to Gemfile
2. Configure: configs/efs values
3. Deploy

## Add

Add the blueprint to your lono project's `Gemfile`.

```ruby
gem "efs", git: "git@github.com:boltopspro/efs.git"
```

## Configure

First, you want to configure the [configs](https://lono.cloud/docs/core/configs/) files. Use [lono seed](https://lono.cloud/reference/lono-seed/) to configure starter values quickly.

    LONO_ENV=development lono seed efs

The generated files in `config/efs` folder look something like this:

    configs/efs/
    ├── params
    │   └── development.txt
    └── variables
        └── development.rb

Here's an example of the parameters.

configs/efs/params/development.txt:

    # Parameter Group: AWS::EFS::FileSystem
    SecurityGroups="" # Can be a blank string but then should set VpcId for managed SecurityGroup created # (required)
    # Encrypted=true
    # IpAddress=
    # KmsKeyId=
    # PerformanceMode=
    # ProvisionedThroughputInMibps=
    # ThroughputMode=

    # Parameter Group: AWS::EC2::SecurityGroup
    # VpcId= # vpc-111 # Should be set if SecurityGroups is not set (empty string)

configs/efs/variables/development.rb:

    # Create EFS Mount Targets in whichever subnets you need them
    @subnets = %w[subnet-111 subnet-222 subnet-333]

## Deploy

Use the [lono cfn deploy](http://lono.cloud/reference/lono-cfn-deploy/) command to deploy. Example:

    lono cfn deploy efs --sure

## Configure Details

### Mounting EFS Volume

To mount the EFS filesystem, you can use the [boltopspro/ec2](https://github.com/boltopspro-docs/ec2) blueprint to launch 2 test instances.  Make sure to:

* Set the `VpcId` and `SubnetId` parameter in the ec2 blueprint configs to values in the same subnet that the EFS have mount targets in.
* Also, make sure to open up the EFS security group, so it allows access from these EC2 instances.  A simple, way is to whitelist the VPC CIDR range.

After the EC2 instances launched, SSH into the instances and mount the volume. Here's an example where the EFS FileSystem Id is `fs-5f83c0f5`:

    sudo yum install -y amazon-efs-utils
    sudo mkdir -p /mnt/efs
    sudo mount -t efs fs-5f83c0f5:/ /mnt/efs

AWS docs: [Mounting EFS File Systems](https://docs.aws.amazon.com/efs/latest/ug/mounting-fs.html)

To automatically mount the volume on a reboot, you can add this line to the `/etc/fstab` file.

/etc/fstab:

    fs-5f83c0f5:/ /mnt/efs efs _netdev,tls,iam 0 0

Before rebooting, test that the mount will work just in case:

    sudo mount -fav

AWS docs: [Mounting Your Amazon EFS File System Automatically](https://docs.aws.amazon.com/efs/latest/ug/mount-fs-auto-mount-onreboot.html#mount-fs-auto-mount-update-fstab)

### DeletionPolicy

The default DeletionPolicy is Retain for safety reasons since most of the properties for the [EFS FileSystem](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-efs-filesystem.html) require replacement. You can override this with the `@deletion_policy` variable.  Example:

configs/efs/variables/development.rb:

```ruby
@deletion_policy = "Delete"
```

### Security Groups

To assign existing security groups to the File System use `SecurityGroups`. Example:

configs/efs/params/development.txt:

    SecurityGroups=sg-111,sg-222

If not set, then the blueprint will create a managed Security Group and assign to it to the File System instance.

### Managed Security Group Rules

To open security group rules on the Managed Security Group, you can use the `@security_group_ingress` variable. Example:

configs/ec2/variables/development.rb:

```ruby
@security_group_ingress = [{
  CidrIp: "10.10.0.0/16", # Example
  FromPort: 2049,
  IpProtocol: "tcp",
  ToPort: 2049,
}]
```
