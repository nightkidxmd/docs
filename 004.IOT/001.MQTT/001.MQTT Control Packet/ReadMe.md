

## MQTT Control Packet of v3.1.1

@(mqtt,iot)

[TOC]

### 1. 名词解释

| 简写 | 说明                 |
| ---- | -------------------- |
| MCP  | MQTT Control Packets |
| FH   | Fixed Header         |
| VH   | Variable header      |



### 2. MCP格式

|  MCP 结构 |
| :----: |
| FH, persent in all MCP|
| VH,present in some MCP|
|Payload, present in some MCP|

#### 2.1 FH格式

|  Bit   |    7     |    6     |    5     |    4     |   3   |   2   |   1   |   0   |
| :----: | :------: | :------: | :------: | :------: | :---: | :---: | :---: | :---: |
| byte 1 | MCP type | MCP type | MCP type | MCP type | FLAGS | FLAGS | FLAGS | FLAGS |
| byte 2 |          |          |          |          |       |       |       |       |

##### 2.1.1 MCP Type

> C:Client
>
> S:Server

|                  | Name        | Value | Direction of flow | Description              |
| ---------------- | ----------- | :---: | ----------------- | ------------------------ |
|                  | Reserved    |   0   | Forbidden         | Reserved                 |
|                  | Reserved    |  15   | Forbidden         | Reserved                 |
| **Connection**   | CONNECT     |   1   | C -> S            | C req to connect to S    |
|                  | CONNACK     |   2   | S -> C            | Connect ack              |
|                  | DISCONNECT  |  14   | C -> S            | C is disconnecting       |
| **Publishment**  | PUBLISH     |   3   | C <=> S           | Publish msg              |
|                  | PUBACK      |   4   | C <=>S            | Publish ack      |
|                  | PUBREC      |   5   | C <=>S            | Publish received |
|                  | PUBREL      |   6   | C <=>S            | Publish release  |
|                  | PUBCOMP     |   7   | C <=>S            | Publish Completed        |
| **Subscription** | SUBSCRIBE   |   8   | C -> S            | C subscribe request      |
|                  | SUBACK      |   9   | S -> C            | Subscribe ack            |
|                  | UNSUBSCRIBE |  10   | C -> S            | Unsubscribe request      |
|                  | UNSUBACK    |  11   | S -> C            | Unsubscribe ack          |
| **Ping**         | PINGREQ     |  12   | C ->S             | Ping req                 |
|                  | PINGRESP    |  13   | S ->C             | Ping resp                |

### 2.2 Connection Sequence

#### 2.2.1 Connect

```sequence
Client -> Server:CONNECT
Server -> Client:CONNACK
```
#### 2.2.2 Disconnect

```sequence
Client -> Server:DISCONNECT
```
### 2.3 Subsciption Sequence

#### 2.3.1 Subscribe

```sequence
Client -> Server:SUBSCRIBE
Server -> Client:SUBACK
```
#### 2.3.2 Unsubscribe

```sequence
Client -> Server:UNSUBSCRIBE
Server -> Client:UNSUBACK
```
### 2.4 Publish Sequence

> Client send msg to Server

#### 2.4.1 QoS 0

```sequence
Client -> Server:PUBLISH
```
#### 2.4.2 QoS 1

```sequence
Client -> Server:PUBLISH
Server -> Client:PUBACK
Note left of Client:Discard
```
#### 2.4.3 QoS 2

```sequence
Client -> Server:PUBLISH
Note right of Server:Store msg or id
Server -> Client:PUBREC
Note left of Client:Discard and Store PUBREC
Client -> Server:PUBREL
Note right of Server:Check onward id then discard
Server -> Client:PUBCOMP
Note left of Client: Discard
```

