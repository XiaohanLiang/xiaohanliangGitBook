# jdcloud\_network\_interface\_attachment

After you have created network interface, you can attach this network interface to an instance.

### Example Usage 

```bash
resource "jdcloud_network_interface_attachment" "attachment-example"{
  instance_id = "i-example"
  network_interface_id = "port-example"
}
```

### Argument Reference

The following arguments are supported:

* `instance_id` - \(Required\) : The id of instance you would like to associate with
* `network_interface_id` - \(Required\): ****The id of network interface you would like to associate
* `auto_delete` - \(Optional\): If this field is set to true, network interface will be deleted automatically after detaching from instance





