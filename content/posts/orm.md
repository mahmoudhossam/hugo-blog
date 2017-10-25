---
title: "Django ORM Problems"
date: 2017-10-25T21:38:48+02:00
draft: false
---

I had a problem at work with a pattern that I think is common for database backend web applications.


Let's suppose we have a model `Device` that looks like the following:

{{< highlight python >}}
class Device(model.Model):
    DEVICE_TYPES = (
        ('A', 'Android'),
        ('I', 'iOS'),
        ('W', 'Web')
    )
    device_type = models.CharField(max_length=1, choices=DEVICE_TYPES, default='W')
    ios_device = models.OneToOneField(AppleDevice, on_delete=models.CASCADE, null=True)
    android_device = models.OneToOneField(AndroidDevice, on_delete=models.CASCADE, null=True)
    web_device = models.OneToOneField(WebDevice, on_delete=models.CASCADE, null=True)
{{< / highlight >}}

Now let's suppose that I want this device to be one of those three relationships, based on the `device_type` attribute.

So, for example an iOS device would have the device type set to iOS and only the `ios_device` would be a non-null reference.

The only way to do this in Django is to add model level validation in `save()`

The way I did it is by overriding the `clean()` method of the model and having `save()` call it, like so:


{{< highlight python >}}
def clean(self):
    # make sure only one type is not-null
    if len([x for x in [self.ios_device, self.android_device, self.web_device] if x]) != 1:
        raise ValidationError('Device only needs one type')
    # make sure linked device matches the device_type attribute
    if ((self.device_type == 'A' and self.android_device is None) or
        (self.device_type == 'I' and self.ios_device is None) or 
        (self.device_type == 'W' and self.web_device is None)):
        raise ValidationError('Linked device does not match device type')
{{< / highlight >}}

Which is obviously very verbose and error prone, is there a better way to do this?
