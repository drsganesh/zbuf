# ZBUF
Simple binary data transfer protocol to send strictly typed data. <br>
Built mainly for transferring typed data from JS to other languages. <br>
Initial versions will focus on JS and Go. <br>
Uses Little Endian. <br>
Mostly zero copy deserialization. <br>
Supports streaming. Data can be written from different functions.<br>
Rows are supported as well as schema maps. <br>
May not support complex nested objects|Maps. <br>

Encoding slice is slower than JSON (~ x2). <br>
Decoding slice is faster than JSON (~ x2.5 - 5) and CBOR (~ x2 - 3) <br>
Encoding map is takes similar time as JSON and is slower than CBOR (~ x1.4x -1.5x ) <br>
Decoding map is faster than JSON (~ x3) and CBOR (~ 20%). <br>
Strict type assertion can be done from js side. <br>

Considering the performance, it is better to use this for sending data from browser JS to backend. CBOR can be used for sending data from backend to browser. Messagepack may be more performant, but CBOR is a standard. <br>


<hr>

- Components
    - Meta
    - Length info [optional]
    - Data [optional]

<hr>

## Structure
First byte will be meta. <br>


### Meta
Meta is a single byte. <br>

- First and second bit
    - 00: Dual
    - 01: Number
    - 10: Custom: Date, Decimal, Error
    - 11: Variable (string, bytes)


#### Dual (0)
Dual element will not be having a data element length info 

- Third and fourth bits
    - 00: Boolean
    - 01: Nil || Null || Undefined : 0b00010000 (16)
    - 10: Array | Object
    - 11: Any : only for array start


- 5-8 bit : 
    - 0000: (0) : Array start     :   0b00100000 (32)
    - 0001: (1) : Object start    :   0b00100001
    - 0010: (2) : Array end       :   0b00100010 (34)
    - 0011: (3) : Object end      :   0b00100011
    - 0100: (4) : Table start     :   0b00100100 
    - 0101: (5) : Row start       :   0b00100101
    - 0110: (6) : Table end       :   0b00100110
    - 0111: (7) : Row end         :   0b00100111 

- Object: Dual array of key value pairs : will be ordered: [][]
- Table: Multiple sequential arrays, first array will be column name: [[],[]]
- Row: Multiple sequential arrays without column name: [[],[]]
- The closing tag is not a must.

- Eighth bit : for bool
    - 0: False  : 0b00000000
    - 1: True   : 0b00000001 

#### Number (1)
- Third bit
    -0 : int
    -1 : float

- Fourth bit
    -0 : unsigned
    -1 : signed

- Fifth ans sixth bit
    - reserved for actual length

- Seventh and eighth bit
    - 00:  8 bit (0)
    - 01: 16 bit (1)
    - 10: 32 bit (2)
    - 11: 64 bit (3)    

- int  : 0b01010011 : int64
- int8 : 0b01010000
- int16: 0b01010001
- int32: 0b01010010
- int64: 0b01010011 : 83
- uint : 0b01000011 : uint64 : 67
- uint8: 0b01000000
- uint16:0b01000001
- uint32:0b01000010
- uint64:0b01000011
- float32: 0b01110010
- float64: 0b01110011



#### Custom (2)
- Third and fourth bit
    - 00: Date-time  : 0b10000000 : stored as int64 : no length info needed. 
    - 01: Decimal
    - 10: 
    - 11: Error : 0b10110000 : <= 256 characters




#### Variable (3)
- Third and fourth bit
    - 00: String    : 0b11000000 : last two bits will be size of length info
    - 01: Bytes     : 0b11010000
    - 10: 
    - 11:

-  Fifth and sixth bit
    - reserved 

- Seventh and eighth bit
    - Unspecified / file (up to) : determines the bytes of length info
        - 00: (0)  8 bit 
        - 01: (1)16 bit
        - 10: (2)32 bit
        - 11: (3) 64 bit


<hr>
 

#### Array
Can avoid meta bytes of elements in some cases, but keeping it for simplicity <br>


<hr>

## Conventions
The fields (properties of object) can be avoided while sending.<br> 
This can be added in the receiving side. <br>


<hr>

## Examples


```javascript
// JS : Encode Object
    const oType = {
        "a": ["string","string"],
        "b": [undefined,"int16"],
    }

    const oData = {
        "a": "abc",
        "b": 123,
        "c": true
    }


    let enc = zbuf.encodeObject(oData, oType)
    if (enc instanceof Error) {        
        return enc
    }
```

```javascript
// JS : Encode Array
 let arType = []
    let arData = ["abc"]
    arData[1] = ["pqr", "stu"]
    arData[2] = `lorum ipsum . `

    // arData[4] = null
    arData[3] = true
    arData[6] = -3.14
    arType[6] = "float32"
    arData[5] = 123
    arType[5] = "int16"

    let dateIn4000 = new Date(4000, 0, 1);
    const UTMS = dateIn4000.getTime()
   
    arData[7] = UTMS
    arType[7] = "uint64"

    arData[8] = dateIn4000




    let enc = zbuf.encodeArray(arData, arType)
    if (enc instanceof Error) {
            return enc
    }
```



