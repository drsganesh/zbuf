# ZBUF v1
Simple binary data transfer protocol to send strictly typed data. <br>
Built mainly for transferring typed data from JS to other languages. <br>
V1 will focus on JS and Go. <br>
Uses Little Endian. <br>
Mostly zero copy. <br>
Nested arrays and objects are not supported. <br>
Rows are supported as well as schema maps. <br>



Decoding and encoding are faster than JSON, and in many cases faster than flatbuffer <br>
Strict type assertion can be done from js side. <br>
It is schemaless, but schema can be added. <br>


- Components
    - Meta
    - Length info [optional]
    - Data [optional]



## Structure
First byte will be meta. <br>


### Meta
Meta is a single byte. <br>

- First bit
    - 0: Element
    - 1: Array : First two elements will be meta of key, then meta of value. Then even elements will be elements of key and odd elements will be elements of value. <br>


#### Element
- Second and third bit
    - 00: Multi (0)
    - 01: Number (1)
    - 10: Bytes (2) : v2: include string in bytes and separate float and int to reduce size of types like int
    - 11: String (3)

<hr>

#### Multi Element : Any, Bool, nil, undefined and [-1,0,1]
Multi element will not be having a data element length info 

- Fourth and fifth bit
    - 00: Boolean
    - 01: Nil || Null || Undefined : 00100000
    - 10: Error : this can indicate that the data has an error (reserved)
    - 11: Any : only for array start

- Eighth bit
    - 0: False 
    - 1: True 

<hr>

#### Number Element
Number element will not be having a length info. <br>
In JS, if type is not specified, it will be a float64. <br>

- Fourth bit
    - 0: Integer
    - 1: Float

- Fifth bit
    - 0: Unsigned
    - 1: Signed

- Sixth bit
    - reserved

- Seventh and eighth bit
    - 00:  8 bit (0)
    - 01: 16 bit (1)
    - 10: 32 bit (2)
    - 11: 64 bit (3)

<hr>

#### Bytes Element
- Fourth and fifth bit
    - 00: Unspecified
    - 01: DT (date time) UTMS  int64 : no length info needed. 
    - 10: Number : Reserved. no length info needed. 
    - 11: File : reserved. there will be a file meta object and unspecified bytes

- Sixth bit
    - Reserved


- Seventh and eighth bit
    - Unspecified / file (up to) : determines the bytes of length info
        - 00:  8 bit 
        - 01: 16 bit
        - 10: 32 bit
        - 11: 64 bit

    - Number (up to) : no length info needed. 
        - 00: 64 bit decimal: dual int32
        - 01: 128 bit big int
        - 10: 256 bit big int
        - 11: 512 bit big int 



       

<hr>

#### String Element
- Fourth bit
    - 0: UTF8
    - 1: UTF16 : reserved

- Fifth  and sixth bit
    - reserved

- Seventh and eighth bit (up to) : determines the bytes of length info
    - 00: 8 bit : up to 255 chars
    - 01: 16 bit :  up to 65535 chars
    - 10: 32 bit : up to 4294967295 chars
    - 11: 64 bit : up to 18446744073709551615 chars

 
<hr>  

#### Array
Can avoid meta bytes of elements in some cases, but keeping it for simplicity <br>


<hr>

## Conventions
The fields (properties of object) can be avoided while sending.<br> 
This can be added in the receiving side. <br>




