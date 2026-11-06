# Toucan Azoteq driver

Vendored from https://github.com/beekeeb/zmk_driver_azoteq at
52fcfe5026c9f74d611d05b2084548e046a7e274, under the included MIT license.
The root zephyr/module.yml registers this driver and the root dts bindings;
config/west.yml intentionally does not also fetch the upstream driver.

Local fixes:

- Run report processing and the ready watchdog on a dedicated work queue.
  Input delivery uses blocking queue writes, and the input consumer forwards
  reports through Bluetooth. Keeping the producer off the system work queue
  lets Bluetooth and keyboard work continue when input delivery backs up.
- Close the I2C communication window and release the device semaphore before
  reporting input, so Bluetooth backpressure cannot hold the device lock.
- Initialize the work item and semaphore before enabling RDY interrupts.
- Finish diagnostic register reads before enabling interrupts, avoiding concurrent
  I2C access between initialization and the event worker.
- Check RDY every 100 ms while awake. If a rising edge was missed, submit the
  normal read handler to service the outstanding communication window. This
  recovers actual movement and touch-up reports without timing out a stationary
  finger or forcing the mouse layer off during a valid touch.

The watchdog checks only GPIO and does not poll I2C while RDY is inactive.
It skips reads while suspended. It adds a periodic CPU wakeup, with a potential
battery cost. It does not recover an electrically stuck I2C bus or a blocked
driver work queue.

Hardware validation: move and lift repeatedly, hold a stationary finger, use
mouse dragging/gestures, and repeat after idle and sleep/wake. Verify the MOU
layer exits on lift and the cursor continues responding. Both the right-half and
Studio-enabled left-half builds passed with ZMK v0.3 and Zephyr SDK 0.16.9.
Hardware reproduction is still needed to confirm this addresses the reported freeze.
