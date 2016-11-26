
Internet Engineering Task Force (IETF)					R. Fielding, Ed.  
Request for Comments: 7233                                         Adobe  
Obsoletes: 2616                                            Y. Lafon, Ed.  
Category: Standards Track                                            W3C  
ISSN: 2070-1721                                          J. Reschke, Ed.  
                                                              greenbytes  
                                                              June 2014  

         Hypertext Transfer Protocol (HTTP/1.1): Range Requests
Abstract
   The Hypertext Transfer Protocol (HTTP) is a stateless application-
   level protocol for distributed, collaborative, hypertext information
   systems.  This document defines range requests and the rules for
   constructing and combining responses to those requests.  
Status of This Memo

   This is an Internet Standards Track document.

   This document is a product of the Internet Engineering Task Force
   (IETF).  It represents the consensus of the IETF community.  It has
   received public review and has been approved for publication by the
   Internet Engineering Steering Group (IESG).  Further information on
   Internet Standards is available in Section 2 of RFC 5741.

   Information about the current status of this document, any errata,
   and how to provide feedback on it may be obtained at
   http://www.rfc-editor.org/info/rfc7233.




Fielding, et al.             Standards Track                    [Page 1]  
RFC 7233                 HTTP/1.1 Range Requests               June 2014  


Copyright Notice

   Copyright (c) 2014 IETF Trust and the persons identified as the
   document authors.  All rights reserved.

   This document is subject to BCP 78 and the IETF Trust's Legal
   Provisions Relating to IETF Documents
   (http://trustee.ietf.org/license-info) in effect on the date of
   publication of this document.  Please review these documents
   carefully, as they describe your rights and restrictions with respect
   to this document.  Code Components extracted from this document must
   include Simplified BSD License text as described in Section 4.e of
   the Trust Legal Provisions and are provided without warranty as
   described in the Simplified BSD License.

   This document may contain material from IETF Documents or IETF
   Contributions published or made publicly available before November
   10, 2008.  The person(s) controlling the copyright in some of this
   material may not have granted the IETF Trust the right to allow
   modifications of such material outside the IETF Standards Process.
   Without obtaining an adequate license from the person(s) controlling
   the copyright in such materials, this document may not be modified
   outside the IETF Standards Process, and derivative works of it may
   not be created outside the IETF Standards Process, except to format
   it for publication as an RFC or to translate it into languages other
   than English.

























Fielding, et al.             Standards Track                    [Page 2]  

RFC 7233                 HTTP/1.1 Range Requests               June 2014   


Table of Contents

   1. Introduction ....................................................4  
      1.1. Conformance and Error Handling .............................4  
      1.2. Syntax Notation ............................................4  
   2. Range Units .....................................................5  
      2.1. Byte Ranges ................................................5  
      2.2. Other Range Units ..........................................7  
      2.3. Accept-Ranges ..............................................7  
   3. Range Requests ..................................................8  
      3.1. Range ......................................................8  
      3.2. If-Range ...................................................9  
   4. Responses to a Range Request ...................................10  
      4.1. 206 Partial Content .......................................10  
      4.2. Content-Range .............................................12  
      4.3. Combining Ranges ..........................................14  
      4.4. 416 Range Not Satisfiable .................................15  
   5. IANA Considerations ............................................16  
      5.1. Range Unit Registry .......................................16  
           5.1.1. Procedure ..........................................16  
           5.1.2. Registrations ......................................16  
      5.2. Status Code Registration ..................................17  
      5.3. Header Field Registration .................................17  
      5.4. Internet Media Type Registration ..........................17  
           5.4.1. Internet Media Type multipart/byteranges ...........18  
   6. Security Considerations ........................................19  
      6.1. Denial-of-Service Attacks Using Range .....................19  
   7. Acknowledgments ................................................19  
   8. References .....................................................20  
      8.1. Normative References ......................................20  
      8.2. Informative References ....................................20  
   Appendix A. Internet Media Type multipart/byteranges ..............21  
   Appendix B. Changes from RFC 2616 .................................22  
   Appendix C. Imported ABNF .........................................22  
   Appendix D. Collected ABNF ........................................23  
   Index .............................................................24  



Fielding, et al.             Standards Track                    [Page 3]
RFC 7233                 HTTP/1.1 Range Requests               June 2014


1.  Introduction

   Hypertext Transfer Protocol (HTTP) clients often encounter
   interrupted data transfers as a result of canceled requests or
   dropped connections.  When a client has stored a partial
   representation, it is desirable to request the remainder of that
   representation in a subsequent request rather than transfer the
   entire representation.  Likewise, devices with limited local storage
   might benefit from being able to request only a subset of a larger
   representation, such as a single page of a very large document, or
   the dimensions of an embedded image.  
   HTTP客户端经常会因为请求取消或者连接断开导致数据传输中断。当用户端已经有缓存了一部分内容的时候，用户端更希望能够请求剩下的内容而不是重新请求完整的内容。而且，对于本机缓存有限制的设备，如果可以只请求大文件的其中的一小部分内容，比如一个非常大的文档的一个单页，或者其中的内嵌的图片将会有很大益处。

   This document defines HTTP/1.1 range requests, partial responses, and
   the multipart/byteranges media type.  Range requests are an OPTIONAL
   feature of HTTP, designed so that recipients not implementing this
   feature (or not supporting it for the target resource) can respond as
   if it is a normal GET request without impacting interoperability.
   Partial responses are indicated by a distinct status code to not be
   mistaken for full responses by caches that might not implement the
   feature.  
   这个文档对HTTP/1.1的Range请求，分段响应以及multipart/byteranges的媒体类型做了定义。Range请求是个可选的HTTP特性，这么设计是因为有些用户端没有应用这个特性（或者说请求的目标不支持Range请求），保证他们可以像正常的请求一样响应而不会影响交互。分段响应通过状态码(206)来标明，不支持分段响应的缓存服务器会响应完整内容。
   Although the range request mechanism is designed to allow for
   extensible range types, this specification only defines requests for
   byte ranges.  
   尽管Range请求机制被设计用来允许可扩展的范围类型，本文档只定义了字节范围的请求。

	
1.1.  Conformance and Error Handling

   The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
   "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this
   document are to be interpreted as described in [RFC2119].

   Conformance criteria and considerations regarding error handling are
   defined in Section 2.5 of [RFC7230].

1.2.  Syntax Notation

   This specification uses the Augmented Backus-Naur Form (ABNF)
   notation of [RFC5234] with a list extension, defined in Section 7 of
   [RFC7230], that allows for compact definition of comma-separated
   lists using a '#' operator (similar to how the '*' operator indicates
   repetition).  Appendix C describes rules imported from other
   documents.  Appendix D shows the collected grammar with all list
   operators expanded to standard ABNF notation.


2.  Range Units

   A representation can be partitioned into subranges according to
   various structural units, depending on the structure inherent in the
   representation's media type.  This "range unit" is used in the
   Accept-Ranges (Section 2.3) response header field to advertise
   support for range requests, the Range (Section 3.1) request header
   field to delineate the parts of a representation that are requested,
   and the Content-Range (Section 4.2) payload header field to describe
   which part of a representation is being transferred.  
  内容可以根据自身的媒体类型的内在结构被分段成区间。在响应头部Accept-Ranges中，“Range unit”用来声明支持range请求，请求头部Range用来描述请求的内容部分，Content-Range报文字段指明了传输了哪一部分文件内容。

     range-unit       = bytes-unit / other-range-unit

2.1.  Byte Ranges

   Since representation data is transferred in payloads as a sequence of
   octets, a byte range is a meaningful substructure for any
   representation transferable over HTTP (Section 3 of [RFC7231]).  The
   "bytes" range unit is defined for expressing subranges of the data's
   octet sequence.  
  由于内容是通过一系列的八位位组报文进行传输的，所以通过HTTP协议传输的任何字节都是有意义的。单位“字节”用来描述数据八进制序列的子范围。

     bytes-unit       = "bytes"

   A byte-range request can specify a single range of bytes or a set of
   ranges within a single representation.  
   byte-range请求可以用来分配单一的字节范围，或者一系列的范围。

     byte-ranges-specifier = bytes-unit "=" byte-range-set
     byte-range-set  = 1#( byte-range-spec / suffix-byte-range-spec )
     byte-range-spec = first-byte-pos "-" [ last-byte-pos ]
     first-byte-pos  = 1*DIGIT
     last-byte-pos   = 1*DIGIT

   The first-byte-pos value in a byte-range-spec gives the byte-offset
   of the first byte in a range.  The last-byte-pos value gives the
   byte-offset of the last byte in the range; that is, the byte
   positions specified are inclusive.  Byte offsets start at zero.
   在byte-range-spec中，first-byte-pos标明Range请求第一个字节的字节偏移量，last-byte-pos标明range请求的最后一个字节的字节偏移量。通过这种方式，包含指明了字节位置。字节偏移量从0开始。

   Examples of byte-ranges-specifier values:  
   byte-ranges-specifier案例：

   o  The first 500 bytes (byte offsets 0-499, inclusive):  
   第一个500字节(字节偏移0-499，包含)  

        bytes=0-499

   o  The second 500 bytes (byte offsets 500-999, inclusive):  
   第二个500字节（字节偏移 500-999，包含）  

        bytes=500-999


   A byte-range-spec is invalid if the last-byte-pos value is present
   and less than the first-byte-pos.  
   如果last-byte-pos的值提供，并且小于the first-byte-pos的值，那么byte-range-spec是无效的。

   A client can limit the number of bytes requested without knowing the
   size of the selected representation.  If the last-byte-pos value is
   absent, or if the value is greater than or equal to the current
   length of the representation data, the byte range is interpreted as
   the remainder of the representation (i.e., the server replaces the
   value of last-byte-pos with a value that is one less than the current
   length of the selected representation).  
   在不知道请求的内容的大小的时候，客户端可以限制请求的字节大小。如果last-byte-pos的值没有提供，或者提供的值大于等于目标内容的大小，那么字节范围会被认为是内容的剩下部分（i.e. 服务端用小于请求内容长度的值替换last-byte-pos的值）。  

   A client can request the last N bytes of the selected representation
   using a suffix-byte-range-spec.  
   用户端可以用suffix-byte-range-spec来请求内容的最后N个字节。  

     suffix-byte-range-spec = "-" suffix-length
     suffix-length = 1*DIGIT

   If the selected representation is shorter than the specified
   suffix-length, the entire representation is used.  
   如果请求文件比指定的suffix-length值小，那么将请求整个文件。  

   Additional examples, assuming a representation of length 10000:  
   另外的例子，假设一个1000字节大小的文件：

   o  The final 500 bytes (byte offsets 9500-9999, inclusive):  
   最后的500字节（字节偏移 9500-9999 ，包含）：  

        bytes=-500

   Or:

        bytes=9500-

   o  The first and last bytes only (bytes 0 and 9999):  
   只请求第一个和最后一个字节（字节0 和 9999）：  

        bytes=0-0,-1

   o  Other valid (but not canonical) specifications of the second 500  
   其他合法（但不规范）的第二个500字节说明：  
      bytes (byte offsets 500-999, inclusive):

        bytes=500-600,601-999
        bytes=500-700,601-999

   If a valid byte-range-set includes at least one byte-range-spec with
   a first-byte-pos that is less than the current length of the
   representation, or at least one suffix-byte-range-spec with a
   non-zero suffix-length, then the byte-range-set is satisfiable.
   Otherwise, the byte-range-set is unsatisfiable.  
   如果一个有效的byte-range-set至少包含一个first-byte-pos的值小于文件当前长度的byte-range-spec，或者至少包含一个suffix-length的值非0的suffix-byte-range-spec，那么这个byte-range-set是可以满足的，否则，不满足要求。   


   In the byte-range syntax, first-byte-pos, last-byte-pos, and
   suffix-length are expressed as decimal number of octets.  Since there
   is no predefined limit to the length of a payload, recipients MUST
   anticipate potentially large decimal numerals and prevent parsing
   errors due to integer conversion overflows.  
   在byte-range语法中，第一字节位置、最后字节位置和词尾长度是通过十进制数字来表示八进制的。因为对报文的程度并没有预先做限制，所以接收端必须预判到可能的大的十进制数字，并且避免因为整数转换溢出而产生错误。
   

2.2.  Other Range Units

   Range units are intended to be extensible.  New range units ought to
   be registered with IANA, as defined in Section 5.1.  
   范围单位的目的是可以扩展的。新的范围单位必须根据5.1章节定义的向IANA进行注册。  

     other-range-unit = token

2.3.  Accept-Ranges

   The "Accept-Ranges" header field allows a server to indicate that it
   supports range requests for the target resource.  
   "Accept-Ranges"响应头部标明服务器支持range请求。

     Accept-Ranges     = acceptable-ranges
     acceptable-ranges = 1#range-unit / "none"

   An origin server that supports byte-range requests for a given target
   resource MAY send  
   源服务器如果支持字节范围的range请求，可能会发送

     Accept-Ranges: bytes

   to indicate what range units are supported.  A client MAY generate
   range requests without having received this header field for the
   resource involved.  Range units are defined in Section 2.
   头部标明支持的范围单位。客户端在没有收到这个头部的时候，可能会发起range请求去获取资源。范围单位在Section2定义。  

   A server that does not support any kind of range request for the
   target resource MAY send  

     Accept-Ranges: none

   to advise the client not to attempt a range request.  
   服务器如果不支持任何范围请求，可能会发送 Accept-Ranges: none 头部建议客户端不要尝试range请求。



3.  Range Requests

3.1.  Range

   The "Range" header field on a GET request modifies the method
   semantics to request transfer of only one or more subranges of the
   selected representation data, rather than the entire selected
   representation data.  
   GET请求头部中的“Range”头部变更了方法语义，从请求完整文件变成只请求一个或者多个子范围的文件内容。  

     Range = byte-ranges-specifier / other-ranges-specifier
     other-ranges-specifier = other-range-unit "=" other-range-set
     other-range-set = 1*VCHAR

   A server MAY ignore the Range header field.  However, origin servers
   and intermediate caches ought to support byte ranges when possible,
   since Range supports efficient recovery from partially failed
   transfers and partial retrieval of large representations.  A server
   MUST ignore a Range header field received with a request method other
   than GET.  
   服务器可能会忽略Range头部。但是，如果可能源服务器和中间缓存服务器应当支持字节范围请求，以便于快速的从分段传输失败中恢复，以及提升大文件的分部分请求效率。对于其他的非GET请求，服务器应该忽略Range头部。

   An origin server MUST ignore a Range header field that contains a
   range unit it does not understand.  A proxy MAY discard a Range
   header field that contains a range unit it does not understand.  
   如果Range请求包含的范围单位不能被理解，那么服务器必须忽略Range请求，代理可服务器能会丢弃Range头部。

   A server that supports range requests MAY ignore or reject a Range
   header field that consists of more than two overlapping ranges, or a
   set of many small ranges that are not listed in ascending order,
   since both are indications of either a broken client or a deliberate
   denial-of-service attack (Section 6.1).  A client SHOULD NOT request
   multiple ranges that are inherently less efficient to process and
   transfer than a single range that encompasses the same data.  
   如果Range头部包含两个互相套圈的范围、或者不是按照升序顺序排列的一系列众多小范围请求，支持Range请求的服务器可能会忽略或者拒绝这个Range头部字段，因为这两种情况可能是客户端异常，或者恶意的拒绝服务攻击（Section 6.1）.客户端尽量不要一次发起多个Range请求，因为对于同一数据，多个range请求的处理效率比一个range请求效率更低。  

   A client that is requesting multiple ranges SHOULD list those ranges
   in ascending order (the order in which they would typically be
   received in a complete representation) unless there is a specific
   need to request a later part earlier.  For example, a user agent
   processing a large representation with an internal catalog of parts
   might need to request later parts first, particularly if the
   representation consists of pages stored in reverse order and the user
   agent wishes to transfer one page at a time.  
   在请求多个范围的时候，除非有特殊的需要先请求一个靠后的部分，不然客户端应该按照升序的方式排列这些范围（按照请求完整文件时客户端收到的顺序）。比如，用户端在处理大文件的时候，需要先处理内部的目录部分，特别是文件内容是以倒序的方式保存而用户希望先请求其中的一个页面。
   

   The Range header field is evaluated after evaluating the precondition
   header fields defined in [RFC7232], and only if the result in absence
   of the Range header field would be a 200 (OK) response.  In other
   words, Range is ignored when a conditional GET would result in a 304
   (Not Modified) response.  
   Range请求头部在【RFC7232】定义的前置条件被评估后才会被评估，并且需要在缺少Range头部的时候响应200状态码。换句话说，一个会导致304响应的有条件的GET请求，会导致Range请求被忽略。


   The If-Range header field (Section 3.2) can be used as a precondition
   to applying the Range header field.  
   If-Range（Section 3.2）可以用来作为Range请求的的前置条件。

   If all of the preconditions are true, the server supports the Range
   header field for the target resource, and the specified range(s) are
   valid and satisfiable (as defined in Section 2.1), the server SHOULD
   send a 206 (Partial Content) response with a payload containing one
   or more partial representations that correspond to the satisfiable
   ranges requested, as defined in Section 4.  
   如果所有的前置条件是真的，服务器支持Range请求头部，并且请求的范围是有效并且可以满足的，服务器应该发送206响应，给用户发送他们请求的满足条件的文件范围内容。

   If all of the preconditions are true, the server supports the Range
   header field for the target resource, and the specified range(s) are
   invalid or unsatisfiable, the server SHOULD send a 416 (Range Not
   Satisfiable) response.  
   如果所有前置条件是真的，服务器也支持Range请求头部，但是指定的范围是有效或者无法满足的，服务器应该发送416（Range Not Satisfiable）响应。

3.2.  If-Range

   If a client has a partial copy of a representation and wishes to have
   an up-to-date copy of the entire representation, it could use the
   Range header field with a conditional GET (using either or both of
   If-Unmodified-Since and If-Match.)  However, if the precondition
   fails because the representation has been modified, the client would
   then have to make a second request to obtain the entire current
   representation.  
   如果客户端已经有一部分文件的副本了，并且希望获取最新的完整文件内容，客户端可以在Range请求头部中使用有条件的GET请求（If-Unmodified-Since/If-Match 择一或者两者都用）。但是，如果因为文件内容已经更新了导致前置条件请求失败，那么客户端应该发起第二个请求去获取当前的完整内容。

   The "If-Range" header field allows a client to "short-circuit" the
   second request.  Informally, its meaning is as follows: if the
   representation is unchanged, send me the part(s) that I am requesting
   in Range; otherwise, send me the entire representation.  
   “If-Range”头部允许客户端“短路”第二个请求。通俗的讲，这个头部的含义：如果请求的内容没有改变，那么将我请求的部分用分段的形式发给我；否则，给我发送完整的文件。
   

     If-Range = entity-tag / HTTP-date

   A client MUST NOT generate an If-Range header field in a request that
   does not contain a Range header field.  A server MUST ignore an
   If-Range header field received in a request that does not contain a
   Range header field.  An origin server MUST ignore an If-Range header
   field received in a request for a target resource that does not
   support Range requests.  
   客户端在没有包含Range头部的时候，请求禁止生成If-Range头部。如果服务器接收到的请求没有包含Range头部，必须忽略If-Range头部；源服务器如果不支持Range请求，那么必须忽略请求头中的If-Range头部。

   A client MUST NOT generate an If-Range header field containing an
   entity-tag that is marked as weak.  A client MUST NOT generate an
   If-Range header field containing an HTTP-date unless the client has
   no entity-tag for the corresponding representation and the date is a
   strong validator in the sense defined by Section 2.2.2 of [RFC7232].  
   如果响应头中有Etag标签并且是弱Etag，那么客户端不能发起If-Range头部。如果响应头包含有HTTP-data信息，客户端也不能发起If-Range头部，除非文件响应头没有Etag头部并且date符合RFC7232 Section2.2.2定义的强验证器说明。

   A server that evaluates an If-Range precondition MUST use the strong
   comparison function when comparing entity-tags (Section 2.3.2 of
   [RFC7232]) and MUST evaluate the condition as false if an HTTP-date
   validator is provided that is not a strong validator in the sense
   defined by Section 2.2.2 of [RFC7232].  A valid entity-tag can be
   distinguished from a valid HTTP-date by examining the first two
   characters for a DQUOTE.  
   服务器在评估校验前置条件If-Range的时候，在比对Etag信息时必须使用强对比功能，并且如果HTTP-date校验器不符合定义的强校验器的说明，必须将条件评估为false。可以通过检查DQUOTE的前两个字符以便从合法的HTTP-date中区别出合法的Etag。


   If the validator given in the If-Range header field matches the
   current validator for the selected representation of the target
   resource, then the server SHOULD process the Range header field as
   requested.  If the validator does not match, the server MUST ignore
   the Range header field.  Note that this comparison by exact match,
   including when the validator is an HTTP-date, differs from the
   "earlier than or equal to" comparison used when evaluating an
   If-Unmodified-Since conditional.  
   如果在If-Range头部中提供的校验值和请求的文件的校验值相匹配，那么服务器应当正常处理Range请求头部。如果两者不匹配，那么服务器必须忽略Range头部。注意：这个比较需要完全匹配，包括当校验值是HTTP-date的情况。

4.  Responses to a Range Request  
   Range请求响应

4.1.  206 Partial Content

   The 206 (Partial Content) status code indicates that the server is
   successfully fulfilling a range request for the target resource by
   transferring one or more parts of the selected representation that
   correspond to the satisfiable ranges found in the request's Range
   header field (Section 3.1).  
   206状态码表明用户发起的Range请求可以满足，并且响应了对应的请求内容。

   If a single part is being transferred, the server generating the 206
   response MUST generate a Content-Range header field, describing what
   range of the selected representation is enclosed, and a payload
   consisting of the range.  For example:  
   如果只有单一的部分传输，206响应的同时，服务器必须响应Content-Range头部，描述响应部分的起始范围，以及这部分范围的报文信息。比如：

     HTTP/1.1 206 Partial Content
     Date: Wed, 15 Nov 1995 06:25:24 GMT
     Last-Modified: Wed, 15 Nov 1995 04:58:08 GMT
     Content-Range: bytes 21010-47021/47022
     Content-Length: 26012
     Content-Type: image/gif

     ... 26012 bytes of partial image data ...

   If multiple parts are being transferred, the server generating the
   206 response MUST generate a "multipart/byteranges" payload, as
   defined in Appendix A, and a Content-Type header field containing the
   multipart/byteranges media type and its required boundary parameter.
   To avoid confusion with single-part responses, a server MUST NOT
   generate a Content-Range header field in the HTTP header section of a
   multiple part response (this field will be sent in each part
   instead).  
   如果传输的是多个部分，服务器必须如附录A定义的方式响应“multipart/byteranges”的206报文，并且Content-Type头部的类型为multipart/byteranges以及对应的边界参数。为了避免和单一传输产生混淆，多部分响应的时候服务器禁止响应Content-Range头部。（这个头部在每一部分单独发送）


   Within the header area of each body part in the multipart payload,
   the server MUST generate a Content-Range header field corresponding
   to the range being enclosed in that body part.  If the selected
   representation would have had a Content-Type header field in a 200
   (OK) response, the server SHOULD generate that same Content-Type
   field in the header area of each body part.  For example:  
   在多部分报文的每个body的头部区域，服务器必须生成一个Content-Range头部对应于这个body部分所属的范围。如果请求的文件响应200的时候有Content-Type头部，多部分响应的时候，服务器应该也在每个body的头部区域生成同样的Content-Type字段。比如：  

     HTTP/1.1 206 Partial Content
     Date: Wed, 15 Nov 1995 06:25:24 GMT
     Last-Modified: Wed, 15 Nov 1995 04:58:08 GMT
     Content-Length: 1741
     Content-Type: multipart/byteranges; boundary=THIS_STRING_SEPARATES

     --THIS_STRING_SEPARATES
     Content-Type: application/pdf
     Content-Range: bytes 500-999/8000

     ...the first range...
     --THIS_STRING_SEPARATES
     Content-Type: application/pdf
     Content-Range: bytes 7000-7999/8000

     ...the second range
     --THIS_STRING_SEPARATES--

   When multiple ranges are requested, a server MAY coalesce any of the
   ranges that overlap, or that are separated by a gap that is smaller
   than the overhead of sending multiple parts, regardless of the order
   in which the corresponding byte-range-spec appeared in the received
   Range header field.  Since the typical overhead between parts of a
   multipart/byteranges payload is around 80 bytes, depending on the
   selected representation's media type and the chosen boundary
   parameter length, it can be less efficient to transfer many small
   disjoint parts than it is to transfer the entire selected
   representation.  
   请求多个范围的时候，服务器可能会将范围重叠的部分合并，或者由小于发送多个部分的所需要头部大小的间隔分开，而不管收到的Range头部中byte-range-spec的顺序是如何的。由于请求文件的媒体类型以及选择的边界参数的长度，多部分字节报文之间的头部大小正常是80字节，分开传输多个小的部分比一次传输所需要的内容效率更低。  

   A server MUST NOT generate a multipart response to a request for a
   single range, since a client that does not request multiple parts
   might not support multipart responses.  However, a server MAY
   generate a multipart/byteranges payload with only a single body part
   if multiple ranges were requested and only one range was found to be
   satisfiable or only one range remained after coalescing.  A client
   that cannot process a multipart/byteranges response MUST NOT generate
   a request that asks for multiple ranges.  
   客户端请求单一部分的时候，服务器禁止响应多部分响应，因为客户端没有请求多部分内容的时候可能是不支持多部分响应的。然而，如果客户端请求多部分内容，请求部分只有一个范围是可满足的或者经过合并后仅剩一个范围，服务器可能会响应只有一个实体部分的多部分/字节范围报文。客户端如果不能处理多部分/字节范围响应，则禁止发起多范围的请求。  

   When a multipart response payload is generated, the server SHOULD
   send the parts in the same order that the corresponding
   byte-range-spec appeared in the received Range header field,
   excluding those ranges that were deemed unsatisfiable or that were
   coalesced into other ranges.  A client that receives a multipart
   response MUST inspect the Content-Range header field present in each
   body part in order to determine which range is contained in that body
   part; a client cannot rely on receiving the same ranges that it
   requested, nor the same order that it requested.  
   当生成多部分响应报文的时候，服务器应该按照接收到的Range字段中byte-range-spec对应的数据发送数据，除非有范围被认为是无法满足的或者被合并到其他范围了。客户端接收到多部分响应后必须检查每个实体部分中的Content-Range头部判断包含了哪些范围；客户端不能依赖于自己发送的顺序认为服务端会响应同样的顺序。

   When a 206 response is generated, the server MUST generate the
   following header fields, in addition to those required above, if the
   field would have been sent in a 200 (OK) response to the same
   request: Date, Cache-Control, ETag, Expires, Content-Location, and
   Vary.  
   响应206的时候，除了上述要求的头部，服务器必须和200响应的时候一样发送以下头部：Date、Cache-Control、Etag、Expires、Content-Location、Vary.（即如果响应200状态码有这些头部，206响应也必须要有这些头部）

   If a 206 is generated in response to a request with an If-Range
   header field, the sender SHOULD NOT generate other representation
   header fields beyond those required above, because the client is
   understood to already have a prior response containing those header
   fields.  Otherwise, the sender MUST generate all of the
   representation header fields that would have been sent in a 200 (OK)
   response to the same request.  
   针对用户请求携带If-Range头部所产生的206响应，发送端不应该生成除了以上要求的头部之外的其他头部，因为客户端被理解为之前的响应已经包含了这些头部。除此之外，发送端必须发送和200响应一样的响应头部。

   A 206 response is cacheable by default; i.e., unless otherwise
   indicated by explicit cache controls (see Section 4.2.2 of
   [RFC7234]).  
   206响应默认是可以缓存的，除非有其他的明确的缓存控制表明不能缓存（参见RFC7234 章节4.2.2）

4.2.  Content-Range

   The "Content-Range" header field is sent in a single part 206
   (Partial Content) response to indicate the partial range of the
   selected representation enclosed as the message payload, sent in each
   part of a multipart 206 response to indicate the range enclosed
   within each body part, and sent in 416 (Range Not Satisfiable)
   responses to provide information about the selected representation.  
   “Content-Range”在单一部分的206响应中表明请求部分范围的起始情况，在多部分的206响应中表明报文中每一部分实体部分涵盖的范围，在416（Range不满足）响应中提供关于请求内容的信息。

     Content-Range       = byte-content-range
                         / other-content-range

     byte-content-range  = bytes-unit SP
                           ( byte-range-resp / unsatisfied-range )

     byte-range-resp     = byte-range "/" ( complete-length / "*" )
     byte-range          = first-byte-pos "-" last-byte-pos
     unsatisfied-range   = "*/" complete-length

     complete-length     = 1*DIGIT

     other-content-range = other-range-unit SP other-range-resp
     other-range-resp    = *CHAR

   （Content-Range: bytes 0-1000/218671）

   If a 206 (Partial Content) response contains a Content-Range header
   field with a range unit (Section 2) that the recipient does not
   understand, the recipient MUST NOT attempt to recombine it with a
   stored representation.  A proxy that receives such a message SHOULD
   forward it downstream.  
   如果206响应中Content-Range头部包含的范围单位（Section2）是接受者不能理解的，接受者禁止尝试将这部分和已经缓存的内容结合。代理服务器接收到这样的信息应该传递给下游设备。

   For byte ranges, a sender SHOULD indicate the complete length of the
   representation from which the range has been extracted, unless the
   complete length is unknown or difficult to determine.  An asterisk
   character ("*") in place of the complete-length indicates that the
   representation length was unknown when the header field was
   generated.  
   对于字节范围请求，发送器应该说明请求内容的完整大小，除非完整大小不知道或者难以确认。这个时候用星号(*)代替完整长度来说明长度未知。

   The following example illustrates when the complete length of the
   selected representation is known by the sender to be 1234 bytes:  
   下面的例子说明了发送者知道请求内容的完整长度是1234字节：  

     Content-Range: bytes 42-1233/1234

   and this second example illustrates when the complete length is
   unknown:  
   第二个例子说明完整长度不知道：  

     Content-Range: bytes 42-1233/*

   A Content-Range field value is invalid if it contains a
   byte-range-resp that has a last-byte-pos value less than its
   first-byte-pos value, or a complete-length value less than or equal
   to its last-byte-pos value.  The recipient of an invalid
   Content-Range MUST NOT attempt to recombine the received content with
   a stored representation.  
   在Content-Range字段中，如果byte-range-resp中last-byte-pos的值比first-byte-pos的值小，或者比完整文件大小还大，那么Content-Range字段的值是无效的。接收到无效的Content-Range客户端禁止尝试将接收的内容和存储的内容重合。  

   A server generating a 416 (Range Not Satisfiable) response to a
   byte-range request SHOULD send a Content-Range header field with an
   unsatisfied-range value, as in the following example:  
   byte-range请求导致的416响应应该发送包含不满足范围值的Content-Range头部，如下：

     Content-Range: bytes */1234

   The complete-length in a 416 response indicates the current length of
   the selected representation.  
   416响应中的完整文件大小表明了文件的当前大小。  

   The Content-Range header field has no meaning for status codes that
   do not explicitly describe its semantic.  For this specification,
   only the 206 (Partial Content) and 416 (Range Not Satisfiable) status
   codes describe a meaning for Content-Range.  
   对于没有明确描述语义的状态码，Content-Range头部没有任何意义。在这个规范中，只有206和416状态码描述了Content-Range的含义。


   The following are examples of Content-Range values in which the
   selected representation contains a total of 1234 bytes:  
   以下是一些关于文件内容为1234个字节大小的Content-Range值的案例：

   o  The first 500 bytes:

        Content-Range: bytes 0-499/1234

   o  The second 500 bytes:

        Content-Range: bytes 500-999/1234

   o  All except for the first 500 bytes:

        Content-Range: bytes 500-1233/1234

   o  The last 500 bytes:

        Content-Range: bytes 734-1233/1234

4.3.  Combining Ranges

   A response might transfer only a subrange of a representation if the
   connection closed prematurely or if the request used one or more
   Range specifications.  After several such transfers, a client might
   have received several ranges of the same representation.  These
   ranges can only be safely combined if they all have in common the
   same strong validator (Section 2.1 of [RFC7232]).  
   响应可能由于连接过早的被断开或者请求使用了一个或者多个Range规范导致只传输了内容的一小部分。经过多次类似的传输后，客户端可能收到同一个内容的多个范围内容，这些范围内容只有在他们都有同样的强校验器（RFC7232 Section2.1）的时候才可以被安全的结合。

   A client that has received multiple partial responses to GET requests
   on a target resource MAY combine those responses into a larger
   continuous range if they share the same strong validator.  
   已经收到了文件内容的多个部分的时候，如果这些部分都有同样的强校验值，则客户端可能将这些响应结合成一个大的连续的范围。

   If the most recent response is an incomplete 200 (OK) response, then
   the header fields of that response are used for any combined response
   and replace those of the matching stored responses.  
   如果大部分最新的响应是不完整的200响应，那么在结合响应的时候，这个响应的头部会被用来替换已有缓存响应相匹配的头部。

   If the most recent response is a 206 (Partial Content) response and
   at least one of the matching stored responses is a 200 (OK), then the
   combined response header fields consist of the most recent 200
   response's header fields.  If all of the matching stored responses
   are 206 responses, then the stored response with the most recent
   header fields is used as the source of header fields for the combined
   response, except that the client MUST use other header fields
   provided in the new response, aside from Content-Range, to replace
   all instances of the corresponding header fields in the stored
   response.  
   如果大部分最新响应是206，并且缓存的响应中有一个相匹配的200响应，那么结合的响应头部会由大部分最新的200响应头部组成。如果所有相匹配的缓存响应都是206响应，那么最新的大部分缓存的响应会作为结合响应的来源，除非客户端必须使用新响应中提供的其他头部字段，除了Content-Range外，替换所有缓存响应头部的值。


   The combined response message body consists of the union of partial
   content ranges in the new response and each of the selected
   responses.  If the union consists of the entire range of the
   representation, then the client MUST process the combined response as
   if it were a complete 200 (OK) response, including a Content-Length
   header field that reflects the complete length.  Otherwise, the
   client MUST process the set of continuous ranges as one of the
   following: an incomplete 200 (OK) response if the combined response
   is a prefix of the representation, a single 206 (Partial Content)
   response containing a multipart/byteranges body, or multiple 206
   (Partial Content) responses, each with one continuous range that is
   indicated by a Content-Range header field.  
   组合响应消息体由新响应中的部分内容范围和所选择的响应联合组成。如果联合由内容的全部范围组成，那么客户端必须像这是个完整200响应那样处理联合响应，包含Content-Length头部反应完整长度。否则，客户端必须像如下方式处理这一系列连续的范围：如果联合响应是内容的前面部分，则响应非完整200响应；一个206响应包含多个部分字节范围实体，或者多个206响应，通过Content-Range字段表明每个连续的范围。

4.4.  416 Range Not Satisfiable

   The 416 (Range Not Satisfiable) status code indicates that none of
   the ranges in the request's Range header field (Section 3.1) overlap
   the current extent of the selected resource or that the set of ranges
   requested has been rejected due to invalid ranges or an excessive
   request of small or overlapping ranges.  
   416状态码说明：1. 请求中的Range头部的范围超过了所要请求文件的长度；或者 2. 请求的范围集合因为是无效的范围被拒绝； 或者 3. 过多的请求比较小的或者重复的范围；

   For byte ranges, failing to overlap the current extent means that the
   first-byte-pos of all of the byte-range-spec values were greater than
   the current length of the selected representation.  When this status
   code is generated in response to a byte-range request, the sender
   SHOULD generate a Content-Range header field specifying the current
   length of the selected representation (Section 4.2).  
   对于字节范围，请求与当前长度相同的部分失败意味着所有byte-range-spec的值中，first-byte-pos的值比请求文件的当前长度更大。字节范围请求中生成这个状态码的时候，发送器应该生成一个Content-Range头部表明请求文件的当前长度（Section 4.2）

   For example:

     HTTP/1.1 416 Range Not Satisfiable
     Date: Fri, 20 Jan 2012 15:41:54 GMT
     Content-Range: bytes */47022

      Note: Because servers are free to ignore Range, many
      implementations will simply respond with the entire selected
      representation in a 200 (OK) response.  That is partly because
      most clients are prepared to receive a 200 (OK) to complete the
      task (albeit less efficiently) and partly because clients might
      not stop making an invalid partial request until they have
      received a complete representation.  Thus, clients cannot depend
      on receiving a 416 (Range Not Satisfiable) response even when it
      is most appropriate.  
      注意：因为服务器可以自由忽略Range请求，很多应用会简单的200响应完整的文件内容。一部分是因为大部分客户端都准备接收200响应完成任务（即使效率比较低），一部分是因为客户端在接收到完整的内容前可能不会停止发送无效的分段请求。所以，即使416响应是更适合的，客户端也不能依赖于收到416响应。


5.  IANA Considerations

5.1.  Range Unit Registry

   The "HTTP Range Unit Registry" defines the namespace for the range
   unit names and refers to their corresponding specifications.  The
   registry has been created and is now maintained at
   <http://www.iana.org/assignments/http-parameters>.  
   “HTTP Range Unit Registry” 说明了范围单位名称的命名空间以及相应的规格。通过http://www.iana.org/assignments/http-parameters进行创建和维护说明。  

5.1.1.  Procedure
   流程  
   Registration of an HTTP Range Unit MUST include the following fields:  
   注册一个HTTP范围单位必须包含以下字段：  

   o  Name (名称)  

   o  Description（描述）  

   o  Pointer to specification text（规范文档的指南）  

   Values to be added to this namespace require IETF Review (see
   [RFC5226], Section 4.1).  
   域名空间增加值需要经过IETF组织的复审.  


5.1.2.  Registrations  
   登记  

   The initial range unit registry contains the registrations below:  
   最开始登记的范围单位包含以下注册的：

   +-------------+---------------------------------------+-------------+
   | Range Unit  | Description                           | Reference   |
   | Name        |                                       |             |
   +-------------+---------------------------------------+-------------+
   | bytes       | a range of octets                     | Section 2.1 |
   | none        | reserved as keyword, indicating no    | Section 2.3 |
   |             | ranges are supported                  |             |
   +-------------+---------------------------------------+-------------+

   The change controller is: "IETF (iesg@ietf.org) - Internet
   Engineering Task Force".  
   变更控制机构：IETF(iesg@ietf.org)-互联网工程工作小组


5.2.  Status Code Registration  
   状态码登记  
   The "Hypertext Transfer Protocol (HTTP) Status Code Registry" located
   at <http://www.iana.org/assignments/http-status-codes> has been
   updated to include the registrations below:  

   +-------+-----------------------+-------------+
   | Value | Description           | Reference   |
   +-------+-----------------------+-------------+
   | 206   | Partial Content       | Section 4.1 |
   | 416   | Range Not Satisfiable | Section 4.4 |
   +-------+-----------------------+-------------+
  
   
   
5.3.  Header Field Registration

   HTTP header fields are registered within the "Message Headers"
   registry maintained at
   <http://www.iana.org/assignments/message-headers/>.

   This document defines the following HTTP header fields, so their
   associated registry entries have been updated according to the
   permanent registrations below (see [BCP90]):

   +-------------------+----------+----------+-------------+
   | Header Field Name | Protocol | Status   | Reference   |
   +-------------------+----------+----------+-------------+
   | Accept-Ranges     | http     | standard | Section 2.3 |
   | Content-Range     | http     | standard | Section 4.2 |
   | If-Range          | http     | standard | Section 3.2 |
   | Range             | http     | standard | Section 3.1 |
   +-------------------+----------+----------+-------------+

   The change controller is: "IETF (iesg@ietf.org) - Internet
   Engineering Task Force".

5.4.  Internet Media Type Registration

   IANA maintains the registry of Internet media types [BCP13] at
   <http://www.iana.org/assignments/media-types>.

   This document serves as the specification for the Internet media type
   "multipart/byteranges".  The following has been registered with IANA.


5.4.1.  Internet Media Type multipart/byteranges

   Type name:  multipart

   Subtype name:  byteranges

   Required parameters:  boundary

   Optional parameters:  N/A

   Encoding considerations:  only "7bit", "8bit", or "binary" are
      permitted

   Security considerations:  see Section 6

   Interoperability considerations:  N/A

   Published specification:  This specification (see Appendix A).

   Applications that use this media type:  HTTP components supporting
      multiple ranges in a single request.

   Fragment identifier considerations:  N/A

   Additional information:

      Deprecated alias names for this type:  N/A

      Magic number(s):  N/A

      File extension(s):  N/A

      Macintosh file type code(s):  N/A

   Person and email address to contact for further information:  See
      Authors' Addresses section.

   Intended usage:  COMMON

   Restrictions on usage:  N/A

   Author:  See Authors' Addresses section.

   Change controller:  IESG


6.  Security Considerations

   This section is meant to inform developers, information providers,
   and users of known security concerns specific to the HTTP range
   request mechanisms.  More general security considerations are
   addressed in HTTP messaging [RFC7230] and semantics [RFC7231].

6.1.  Denial-of-Service Attacks Using Range

   Unconstrained multiple range requests are susceptible to denial-of-
   service attacks because the effort required to request many
   overlapping ranges of the same data is tiny compared to the time,
   memory, and bandwidth consumed by attempting to serve the requested
   data in many parts.  Servers ought to ignore, coalesce, or reject
   egregious range requests, such as requests for more than two
   overlapping ranges or for many small ranges in a single set,
   particularly when the ranges are requested out of order for no
   apparent reason.  Multipart range requests are not designed to
   support random access.

7.  Acknowledgments

   See Section 10 of [RFC7230].


8.  References

8.1.  Normative References

   [RFC2046]  Freed, N. and N. Borenstein, "Multipurpose Internet Mail
              Extensions (MIME) Part Two: Media Types", RFC 2046,
              November 1996.

   [RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate
              Requirement Levels", BCP 14, RFC 2119, March 1997.

   [RFC5234]  Crocker, D., Ed. and P. Overell, "Augmented BNF for Syntax
              Specifications: ABNF", STD 68, RFC 5234, January 2008.

   [RFC7230]  Fielding, R., Ed. and J. Reschke, Ed., "Hypertext Transfer
              Protocol (HTTP/1.1): Message Syntax and Routing",
              RFC 7230, June 2014.

   [RFC7231]  Fielding, R., Ed. and J. Reschke, Ed., "Hypertext Transfer
              Protocol (HTTP/1.1): Semantics and Content", RFC 7231,
              June 2014.

   [RFC7232]  Fielding, R., Ed. and J. Reschke, Ed., "Hypertext Transfer
              Protocol (HTTP/1.1): Conditional Requests", RFC 7232,
              June 2014.

   [RFC7234]  Fielding, R., Ed., Nottingham, M., Ed., and J. Reschke,
              Ed., "Hypertext Transfer Protocol (HTTP/1.1): Caching",
              RFC 7234, June 2014.

8.2.  Informative References

   [BCP13]    Freed, N., Klensin, J., and T. Hansen, "Media Type
              Specifications and Registration Procedures", BCP 13,
              RFC 6838, January 2013.

   [BCP90]    Klyne, G., Nottingham, M., and J. Mogul, "Registration
              Procedures for Message Header Fields", BCP 90, RFC 3864,
              September 2004.

   [RFC2616]  Fielding, R., Gettys, J., Mogul, J., Frystyk, H.,
              Masinter, L., Leach, P., and T. Berners-Lee, "Hypertext
              Transfer Protocol -- HTTP/1.1", RFC 2616, June 1999.

   [RFC5226]  Narten, T. and H. Alvestrand, "Guidelines for Writing an
              IANA Considerations Section in RFCs", BCP 26, RFC 5226,
              May 2008.


Appendix A.  Internet Media Type multipart/byteranges

   When a 206 (Partial Content) response message includes the content of
   multiple ranges, they are transmitted as body parts in a multipart
   message body ([RFC2046], Section 5.1) with the media type of
   "multipart/byteranges".  
   当一个206响应信息包含多个范围内容的时候，他们会通过媒体类型为“multipart/byteranges”的一个多部分信息实体被传输。  

   The multipart/byteranges media type includes one or more body parts,
   each with its own Content-Type and Content-Range fields.  The
   required boundary parameter specifies the boundary string used to
   separate each body part.  
   multipart/byteranges 的媒体类型包含一个或者多个实体部分，每个实体有自己的Content-Type和Content-Range字段，通过必须的边界字符串来区分每个实体部分。  

   Implementation Notes:

   1.  Additional CRLFs might precede the first boundary string in the
       body.  
   实体中额外的CRLFs可能会出现在第一个边界字符串中。  

   2.  Although [RFC2046] permits the boundary string to be quoted, some
       existing implementations handle a quoted boundary string
       incorrectly.  
   尽管RFC2046允许边界字符串被包含，但是部分已经存在的应用程序在处理一个被包含的边界字符串的时候会出错。  

   3.  A number of clients and servers were coded to an early draft of
       the byteranges specification that used a media type of multipart/
       x-byteranges, which is almost (but not quite) compatible with
       this type.  
   大量的客户端和服务器使用早期版本草案编码，使用和当前的媒体类型相兼容的的multipart/x-byteranges媒体类型。  

   Despite the name, the "multipart/byteranges" media type is not
   limited to byte ranges.  The following example uses an "exampleunit"
   range unit:  
   除了名称，“multipart/byteranges”媒体类型不仅限于自己范围。下面的例子使用“exampleunit”范围单位：  

     HTTP/1.1 206 Partial Content
     Date: Tue, 14 Nov 1995 06:25:24 GMT
     Last-Modified: Tue, 14 July 04:58:08 GMT
     Content-Length: 2331785
     Content-Type: multipart/byteranges; boundary=THIS_STRING_SEPARATES

     --THIS_STRING_SEPARATES
     Content-Type: video/example
     Content-Range: exampleunit 1.2-4.3/25

     ...the first range...
     --THIS_STRING_SEPARATES
     Content-Type: video/example
     Content-Range: exampleunit 11.2-14.3/25

     ...the second range
     --THIS_STRING_SEPARATES--


Appendix B.  Changes from RFC 2616

   Servers are given more leeway in how they respond to a range request,
   in order to mitigate abuse by malicious (or just greedy) clients.
   (Section 3.1)  
   服务器在如何响应Range请求方面有所偏差，主要应对减轻不良的客户端恶意请求影响。  

   A weak validator cannot be used in a 206 response.  (Section 4.1)  
   在206响应中，不能使用弱的校验值  

   The Content-Range header field only has meaning when the status code
   explicitly defines its use.  (Section 4.2)  
   只有当状态码明确定义了他的用处时Content-Range头部才有意义。  

   This specification introduces a Range Unit Registry.  (Section 5.1)

   multipart/byteranges can consist of a single part.  (Appendix A)

Appendix C.  Imported ABNF

   The following core rules are included by reference, as defined in
   Appendix B.1 of [RFC5234]: ALPHA (letters), CR (carriage return),
   CRLF (CR LF), CTL (controls), DIGIT (decimal 0-9), DQUOTE (double
   quote), HEXDIG (hexadecimal 0-9/A-F/a-f), LF (line feed), OCTET (any
   8-bit sequence of data), SP (space), and VCHAR (any visible US-ASCII
   character).

   Note that all rules derived from token are to be compared
   case-insensitively, like range-unit and acceptable-ranges.

   The rules below are defined in [RFC7230]:

     OWS        = <OWS, see [RFC7230], Section 3.2.3>
     token      = <token, see [RFC7230], Section 3.2.6>

   The rules below are defined in other parts:

     HTTP-date  = <HTTP-date, see [RFC7231], Section 7.1.1.1>
     entity-tag = <entity-tag, see [RFC7232], Section 2.3>


Appendix D.  Collected ABNF

   In the collected ABNF below, list rules are expanded as per Section
   1.2 of [RFC7230].

   Accept-Ranges = acceptable-ranges

   Content-Range = byte-content-range / other-content-range

   HTTP-date = <HTTP-date, see [RFC7231], Section 7.1.1.1>

   If-Range = entity-tag / HTTP-date

   OWS = <OWS, see [RFC7230], Section 3.2.3>

   Range = byte-ranges-specifier / other-ranges-specifier

   acceptable-ranges = ( *( "," OWS ) range-unit *( OWS "," [ OWS
    range-unit ] ) ) / "none"

   byte-content-range = bytes-unit SP ( byte-range-resp /
    unsatisfied-range )
   byte-range = first-byte-pos "-" last-byte-pos
   byte-range-resp = byte-range "/" ( complete-length / "*" )
   byte-range-set = *( "," OWS ) ( byte-range-spec /
    suffix-byte-range-spec ) *( OWS "," [ OWS ( byte-range-spec /
    suffix-byte-range-spec ) ] )
   byte-range-spec = first-byte-pos "-" [ last-byte-pos ]
   byte-ranges-specifier = bytes-unit "=" byte-range-set
   bytes-unit = "bytes"

   complete-length = 1*DIGIT

   entity-tag = <entity-tag, see [RFC7232], Section 2.3>

   first-byte-pos = 1*DIGIT

   last-byte-pos = 1*DIGIT

   other-content-range = other-range-unit SP other-range-resp
   other-range-resp = *CHAR
   other-range-set = 1*VCHAR
   other-range-unit = token
   other-ranges-specifier = other-range-unit "=" other-range-set

   range-unit = bytes-unit / other-range-unit

   suffix-byte-range-spec = "-" suffix-length

   suffix-length = 1*DIGIT

   token = <token, see [RFC7230], Section 3.2.6>

   unsatisfied-range = "*/" complete-length

Index

   2
      206 Partial Content (status code)  10

   4
      416 Range Not Satisfiable (status code)  15

   A
      Accept-Ranges header field  7

   C
      Content-Range header field  12

   G
      Grammar
         Accept-Ranges  7
         acceptable-ranges  7
         byte-content-range  12
         byte-range  12
         byte-range-resp  12
         byte-range-set  5
         byte-range-spec  5
         byte-ranges-specifier  5
         bytes-unit  5
         complete-length  12
         Content-Range  12
         first-byte-pos  5
         If-Range  9
         last-byte-pos  5
         other-content-range  12
         other-range-resp  12
         other-range-unit  5, 7
         Range  8
         range-unit  5
         ranges-specifier  5
         suffix-byte-range-spec  6
         suffix-length  6
         unsatisfied-range  12

   I
      If-Range header field  9

   M
      Media Type
         multipart/byteranges  18, 21
         multipart/x-byteranges  19
      multipart/byteranges Media Type  18, 21
      multipart/x-byteranges Media Type  21

   R
      Range header field  8

Authors' Addresses

   Roy T. Fielding (editor)
   Adobe Systems Incorporated
   345 Park Ave
   San Jose, CA  95110
   USA

   EMail: fielding@gbiv.com
   URI:   http://roy.gbiv.com/


   Yves Lafon (editor)
   World Wide Web Consortium
   W3C / ERCIM
   2004, rte des Lucioles
   Sophia-Antipolis, AM  06902
   France

   EMail: ylafon@w3.org
   URI:   http://www.raubacapeu.net/people/yves/


   Julian F. Reschke (editor)
   greenbytes GmbH
   Hafenweg 16
   Muenster, NW  48155
   Germany

   EMail: julian.reschke@greenbytes.de
   URI:   http://greenbytes.de/tech/webdav/