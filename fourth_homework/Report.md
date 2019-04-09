# Report of the Fourth Homework of Computer Network Course
<center>卓越二班-2016302580264-黎冠延</center>

## P1
### a.
|Destination Address|Interface|
|--|--|
|AddressOf(H3)|3|
### b.
No, because forwarding depends only on the target address.

## P2
### a.
No, because a shared bus can only be used to transmit data from one source at a specific time.

As said on the book:
> If multiple packets arrive to the router at the same time, each at a different input port, all but one must wait since only one packet can cross the bus at a time.

#### b.
No, As said on the book:
> Note also that two packets cannot be forwarded at the same time, even if they have different destination ports, since only one memory read/write can be done at a time over the shared system bus.

### c.
No, since they should be sent to the same port, and according to the words on book:
> a packet being forwarded to an output port will not be blocked from reaching that output port as long as no other packet is currently being forwarded to that output port. However, if two packets from two different input ports are destined to that same output port, then one will have to wait at the input,
since only one packet can be sent over any given bus at a time.

## P3
Suppose delay for a single packet is $D$, then:
### a. 
$$(n - 1)D$$
Since only one packet is sent for a single time.
### b.
$$(n - 1)D$$
The same as `a.`.
### c.
$$0$$
Since the output ports are all different, there exists no interference between each of the transmit action.

## P4
### Min
Needs 3 time slots.
1. X(1) & Y(2)
2. X(2) & Y(3)
3. Z(3)

### Max
Also 3 time slots -- because of the assumption that non-empty input queue is never idle. So, No matter the first time is `X(1) & Y(2)` or `X(1) & Y(3)` the next time still certainly be `X & Y` and lastly the third time must be `Z(3)`.

## P5
### a.
|IP Mode|Interface|
|---|---|
|224.0.0.0 255.192.0.0|0|
|224.64.0.0 255.255.0.0|1|
|224.0.0.0 254.0.0.0|2|
|225.128.0.0 255.127.0.0|3|
|0.0.0.0 0.0.0.0|3|
Using the longest prefix to shadow `224.128.0.0` in Interface `2`.

### b.
1. `3`, because no prefix it has matched, by the 5th one in the table in `a.`.
2. `2`, for the 3rd one in `a.`.
3. `3`, for the 4th one in `a.`.
