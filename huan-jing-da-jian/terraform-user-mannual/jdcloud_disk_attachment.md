# jdcloud\_disk\_attachment

After a disk has been created, you probably need to attach it to an instance

### Example Usage 

```bash
resource "jdcloud_disk_attachment" "disk-attachment-example"{
  instance_id = "i-example"
  disk_id = "vol-example"
}
```

### Argument Reference

The following arguments are supported:

* instance\_id - \(Required\): The id of target instance
* disk\_id - \(Required\): The id of disk
* auto\_delete - \(Optional\): If this field is set to true, disk will be deleted after it has been detached from  instance
* device\_name - \(Optional\) : Specify the logical attachment point , for example, attachment point can be "vba" "vbc" etc. Just to make sure this point is available with no other device using it.





