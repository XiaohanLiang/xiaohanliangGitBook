# jdcloud\_eip\_association

After you have created an EIP, you can attach this EIP with an instance

### Example Usage

```bash
resource "jdcloud_eip_association" "eip-association-example"{
  instance_id = "i-example"
  elastic_ip_id = "fip-example"
}
```

### Argument Reference 

The following arguments are supported:

* `instance_id` - \(Required\) : The id of instance 
* `elastic_ip_id` - \(Required\): The id of EIP



