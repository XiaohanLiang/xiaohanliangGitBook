# Instance - Create Instance Using Image or Template

## Example Usage 

### Using Official Image

#### 

#### Assume now we have a images consists as follows,

* System Disk 
* Data Disk -  namely  "**vdb**"
* Data Disk -  namely "**vdc**"

```text
resource "jdcloud_instance" "vm-1" {
  az = "cn-north-1a"
  instance_name = "my-vm-1"
  instance_type = "c.n1.large"
  image_id = "bba85cab-dfdc-4359-9218-7a2de429dd80"
  password = "${var.vm_password}"
  subnet_id = "${jdcloud_subnet.jd-subnet-1.id}"
  network_interface_name = "veth1"
  primary_ip = "172.16.0.13"
  secondary_ips = [
    "172.16.0.14",
    "172.16.0.15"]
  # secondary_ip_count   = 2
  security_group_ids = [
    "${jdcloud_network_security_group.sg-1.id}"]
  sanity_check = 1

  elastic_ip_bandwidth_mbps = 10
  elastic_ip_provider = "bgp"

  system_disk = {
    disk_category = "local"
    auto_delete = true
    device_name = "vda"
    no_device = true
  }

  data_disk = {
    disk_category = "local"
    auto_delete = true
    device_name = "vdb"
    no_device = true
  }

  data_disk = {
    disk_category = "cloud"
    auto_delete = true
    device_name = "vdc"
    no_device = true

    az = "cn-north-1a"
    disk_name = "vm1-datadisk-1"
    description = "test"
    disk_type = "premium-hdd"
    disk_size_gb = 50
  }
}

```

No\_Device: This parameter is rarely used. Needed only when you would like to create an instance from **image or template**.  For example, now we have a image ,  disks of this image lists as following :

* System Disk 
* Data Disk -  namely  "**vdb**"
* Data Disk -  namely "**vdc**"

Assume now we would like to create an instance using this image but **except data disks.** Configure as following 

