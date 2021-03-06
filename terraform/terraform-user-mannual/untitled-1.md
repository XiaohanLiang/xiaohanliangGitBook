# jdcloud\_eip

Provides a JDCloud elastic IP address

### Example Usage 

```bash
resource "jdcloud_eip" "example"{
  eip_provider = "bgp"
  bandwidth_mbps = 1
}
```

### Argument Reference

The following arguments are supported:

* `eip_provider` - \(Required\): Name of your ip service provider, can be bgp or no\_bgp, according to the region this instance locates at:
  * cn-north-1 : bgp
  * cn-south-1 : bgp or no\_bgp
  * cn-east-1 : bgp or no\_bgp
  * cn-east-2 : bgp
* `bandwidth_mbps` - \(Required\): Specify the bandwidth of your public ip, varify from 1 to 20
* `elastic_ip_address` - \(Optional\): Specify the IP address you would like to see, or a default IP address will be generated

### Attributes Reference

The following attributes are exported:

* `eip_id` :  The id of this elastic IP, can be used to attach/detach from private IP address

### Import 

Existing EIP can be imported to Terraform state by specifying the EIP id:

```text
terraform import jdcloud_eip.example fip-abc123456
```



