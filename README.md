# omen-light
A command line tool to control LED lights for HP Omen machines


# Build
Install hidapi

```
sudo apt install libhidapi-dev
```

then run:
```
g++ -o omen_light omen_light.cc -lhidapi-libusb
```

# Run
Run the tool to see the usage. Run the tool as root to change the settings.

# Risk
This is not an official app, and is only tested on my own Obelisk 875. It may damage the device. Use at your own risk.

# Credit
Inspired by u/ravicc's prior work: https://www.reddit.com/r/HPOmen/comments/nirgwa/a_way_to_change_hp_omen_lighting_with_a_python/


# Python example from Reddit
https://www.reddit.com/r/HPOmen/comments/nirgwa/a_way_to_change_hp_omen_lighting_with_a_python/


```python
import pywinusb.hid as hid


def sample_handler(data):
    print("Raw data: {0}".format(data))


# VID and PID customization changes here...
filter = hid.HidDeviceFilter(vendor_id = 0x103c, product_id = 0x84fd)
hid_device = filter.get_devices()
device = hid_device[0]
device.open()
print(hid_device)
#target_usage = hid.get_full_usage_id(0x00, 0x3f)
target_usage = hid.get_full_usage_id(0xff00, 0x01)
device.set_raw_data_handler(sample_handler)
print(target_usage)


report = device.find_output_reports()

print(report)
print(report[0])

# data to be transmitted from LED Module
buffer = [0x00]*58 # HP LED Module expects 57 bytes + 1 for report ID
buffer[0] = 0x00 # Data 0 - Report ID , Always 0
buffer[1] = 0x00 # data 1 - ******Not sure what it is, does not see to have an impact
buffer[2] = 0x12 # data 2 - ******Not Tested
buffer[3] = 0x07 # data 3 - Change LED rotation type - Breathing (0x06), Color Cycle (0x07), Blinking (0x08), Static (0x01) etc
buffer[4] = 0x01 # data 4 - Custom Color Count - only used for custom colors leave at 0x01
buffer[5] = 0x01 # data 5 -  Custom Color Number - only used for custom colors leave at 0x01
buffer[8] = 0x00 # data 8 - Only for static and custom color - Red Value in hex, Leave at 0x00 for other modes
buffer[9] = 0x00 # data 9 - Only for static and custom color - Green Value in hex, Leave at 0x00 for other modes
buffer[10] = 0x00 # data 10 - Only for static and custom color - Blue Value in hex, Leave at 0x00 for other modes
buffer[48] = 0x64 # data 49 - Changes Brightness 25% (0x19), 50% (0x32), 75% (0x4b), 100% (0x64)
buffer[49] = 0x0a # data 50 - 0x0a or 0x04 for system vitals, constantly changing stuff - leave at 0x0a
buffer[54] = 0x02 # data 54 - LED Module ID - Front is 0x01, Inside LED Bar is 0x02
buffer[55] = 0x01 # data 55 - *****Not Tested
buffer[56] = 0x01 # data 56 - Changes the Theme - Galaxy (0x01), Volcano (0x02), Jungle (0x03), Ocean (0x04)  etc.
buffer[57] = 0x02 # data 57 - Changes the speed Slow (0x01), Medium (0x02), Fast (0x03)


print(" ".join(hex(n) for n in buffer))


report[0].send(buffer)
```
