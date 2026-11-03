# Toucan Azoteq driver

Vendored from https://github.com/beekeeb/zmk_driver_azoteq at
52fcfe5026c9f74d611d05b2084548e046a7e274, under the included MIT license.
The root zephyr/module.yml registers this driver and the root dts bindings;
config/west.yml intentionally does not also fetch the upstream driver.

Local fixes:

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
system work queue.

Hardware validation: move and lift repeatedly, hold a stationary finger, use
mouse dragging/gestures, and repeat after idle and sleep/wake. Verify the MOU
layer exits on lift and the cursor continues responding. A firmware build and
hardware reproduction are needed to confirm this addresses the reported freeze.
