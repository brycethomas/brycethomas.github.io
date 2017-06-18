---
layout: post_page
title: Bluetooth Power Consumption from Transmission Power Alone
---

There's a vague notion that turning Bluetooth off is one way to meaningfully
increase a phone's battery life.  Assuming this is true, I wanted to upper
bound how much of Bluetooth's power consumption could be attributed to the raw
transmission power, as opposed to the incidental power consumption of the
hardware/software itself.

Here are the battery specs of a representative modern smartphone, the Nexus 5X:

| Spec     | Value                 |
| :------- | :-------------------- |
| Capacity | 2,700mAh              |
| Voltage  | 4V                    |
| mWh      | 2,700 * 4 = 10,800mWh |

Bluetooth Low Energy's (BLE)
[maximum transmission power](https://www.bluetooth.org/docman/handlers/downloaddoc.ashx?doc_id=237781)
(page 16) is **10mW**.  With zero incidental power consumption, here's how long
it would take to drain the Nexus 5X battery based on continuous Bluetooth LE
transmission power alone: 10,800 / 10 = 1,080 hours = **45 days**.

In summary, Bluetooth's power consumption on a modern smartphone is hardly
attributable to raw transmission power alone.
