====================
Conversion Functions
====================

During expression evaluations, Velox doesn't implicitly convert expression
arguments to the correct types. If such a conversion is necessary, two provided
conversion functions can be used explicitly cast values to a particular type.

Conversion Functions
--------------------

.. function:: cast(value AS type) -> type

    Explicitly cast a value as a type. This can be used to cast a varchar to a
    numeric value type and vice versa.

.. function:: try_cast(value AS type) -> type

    Like :func:`cast`, but returns null if the cast fails. ``try_cast(x AS type)``
    is different from ``try(cast(x AS type))`` in that ``try_cast`` only suppresses
    errors happening during the casting itself but not those during the evaluation
    of its argument. For example, ``try_cast(x / 0 as double)`` throws a divide-by-0
    error, while ``try(cast(x / 0 as double))`` returns a NULL.

Supported Conversions
---------------------

The supported conversions are listed below, with from-types given at rows and to-types given at columns. Conversions of ARRAY, MAP, and ROW types
are supported if the conversion of their element types are supported. In addition,
supported conversions to/from JSON are listed in :doc:`json`.

.. list-table::
   :widths: 25 25 25 25 25 25 25 25 25 25 25 25 25 25 25 25 25 25 25
   :header-rows: 1

   * -
     - tinyint
     - smallint
     - integer
     - bigint
     - boolean
     - real
     - double
     - varchar
     - varbinary
     - timestamp
     - timestamp with time zone
     - date
     - interval day to second
     - decimal
     - ipaddress
     - ipprefix
     - tdigest
     - qdigest
   * - tinyint
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     -
     -
     -
     -
     -
     - Y
     -
     -
     -
     -
   * - smallint
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     -
     -
     -
     -
     -
     - Y
     -
     -
     -
     -
   * - integer
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     -
     -
     -
     -
     -
     - Y
     -
     -
     -
     -
   * - bigint
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     -
     -
     -
     -
     -
     - Y
     -
     -
     -
     -
   * - boolean
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     -
     -
     -
     -
     -
     - Y
     -
     -
     -
     -
   * - real
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     -
     -
     -
     -
     -
     - Y
     -
     -
     -
     -
   * - double
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     -
     -
     -
     -
     -
     - Y
     -
     -
     -
     -
   * - varchar
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     -
     - Y
     - Y
     - Y
     -
     - Y
     - Y
     - Y
     -
     -
   * - varbinary
     -
     -
     -
     -
     -
     -
     -
     -
     -
     -
     -
     -
     -
     -
     - Y
     - Y
     - Y
     - Y
   * - timestamp
     -
     -
     -
     -
     -
     -
     -
     - Y
     -
     - Y
     - Y
     - Y
     -
     -
     -
     -
     -
     -
   * - timestamp with time zone
     -
     -
     -
     -
     -
     -
     -
     - Y
     -
     - Y
     -
     - Y
     -
     -
     -
     -
     -
     -
   * - date
     -
     -
     -
     -
     -
     -
     -
     - Y
     -
     - Y
     - Y
     -
     -
     -
     -
     -
     -
     -
   * - interval day to second
     -
     -
     -
     -
     -
     -
     -
     - Y
     -
     -
     -
     -
     -
     -
     -
     -
     -
     -
   * - decimal
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     - Y
     -
     -
     -
     -
     -
     - Y
     -
     -
     -
     -
   * - ipaddress
     -
     -
     -
     -
     -
     -
     -
     - Y
     - Y
     -
     -
     -
     -
     -
     -
     - Y
     -
     -
   * - ipprefix
     -
     -
     -
     -
     -
     -
     -
     - Y
     -
     -
     -
     -
     -
     -
     - Y
     - Y
     -
     -
   * - tdigest
     -
     -
     -
     -
     -
     -
     -
     -
     - Y
     -
     -
     -
     -
     -
     -
     -
     -
     -
   * - qdigest
     -
     -
     -
     -
     -
     -
     -
     -
     - Y
     -
     -
     -
     -
     -
     -
     -
     -
     -

Cast to Integral Types
----------------------

Integral types include bigint, integer, smallint, and tinyint.

From integral types
^^^^^^^^^^^^^^^^^^^

Casting one integral type to another is allowed when the input value is within
the range of the result type. Casting from invalid input values throws.

Valid examples:

::

  SELECT cast(1234567 as bigint); -- 1234567
  SELECT cast(12 as tinyint); -- 12

Invalid examples:

::

  SELECT cast(1234 as tinyint); -- Out of range
  SELECT cast(1234567 as smallint); -- Out of range

From floating-point types
^^^^^^^^^^^^^^^^^^^^^^^^^

Casting from floating-point input to an integral type rounds the input value to
the closest integral value. It is allowed when the rounded result is within the
range of the result type. Casting from invalid input values throws.

Valid examples

::

  SELECT cast(12345.12 as bigint); -- 12345
  SELECT cast(12345.67 as bigint); -- 12346
  SELECT cast(127.1 as tinyint); -- 127
  SELECT cast(nan() as integer); -- 0
  SELECT cast(nan() as smallint); -- 0
  SELECT cast(nan() as tinyint); -- 0

Invalid examples

::

  SELECT cast(127.8 as tinyint); -- Out of range
  SELECT cast(1234567.89 as smallint); -- Out of range
  SELECT cast(infinity() as bigint); -- Out of range

Casting NaN to bigint returns 0 in Velox but throws in Presto. We keep the
behavior of Velox by intention because this is more consistent with other
supported cases.

::

  SELECT cast(nan() as bigint); -- 0


From VARCHAR
^^^^^^^^^^^^

Casting a string to an integral type is allowed if the string represents an
integral number within the range of the result type. By default, casting from
strings that represent floating-point numbers is not allowed.
Casting from invalid input values throws.

Valid examples

::

  SELECT cast('12345' as bigint); -- 12345
  SELECT cast('+1' as tinyint); -- 1
  SELECT cast('-1' as tinyint); -- -1

Invalid examples

::

  SELECT cast('12345.67' as tinyint); -- Invalid argument
  SELECT cast('12345.67' as bigint); -- Invalid argument
  SELECT cast('1.2' as tinyint); -- Invalid argument
  SELECT cast('-1.8' as tinyint); -- Invalid argument
  SELECT cast('1.' as tinyint); -- Invalid argument
  SELECT cast('-1.' as tinyint); -- Invalid argument
  SELECT cast('0.' as tinyint); -- Invalid argument
  SELECT cast('.' as tinyint); -- Invalid argument
  SELECT cast('-.' as tinyint); -- Invalid argument

From decimal
^^^^^^^^^^^^

The decimal part is rounded.

Valid examples

::

  SELECT cast(2.56 decimal(6, 2) as integer); -- 3
  SELECT cast(3.46 decimal(6, 2) as integer); -- 3

Invalid examples

::

  SELECT cast(214748364890 decimal(12, 2) as integer); -- Out of range

Cast to Boolean
---------------

From integral and floating-point types
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Casting from integral or floating-point numbers to boolean is allowed. Non-zero
numbers are converted to `true` while zero is converted to `false`.

Valid examples

::

  SELECT cast(1 as boolean); -- true
  SELECT cast(0 as boolean); -- false
  SELECT cast(12 as boolean); -- true
  SELECT cast(-1 as boolean); -- true
  SELECT cast(1.0 as boolean); -- true
  SELECT cast(1.1 as boolean); -- true
  SELECT cast(-1.1 as boolean); -- true
  SELECT cast(nan() as boolean); -- true
  SELECT cast(infinity() as boolean); -- true
  SELECT cast(0.0000000000001 as boolean); -- true
  SELECT cast(0.5 as boolean); -- true
  SELECT cast(-0.5 as boolean); -- true

From VARCHAR
^^^^^^^^^^^^

The strings `t, f, 1, 0, true, false` and their upper case equivalents are allowed to be casted to boolean.
Casting from other strings to boolean throws.

Valid examples

::

  SELECT cast('1' as boolean); -- true
  SELECT cast('0' as boolean); -- false
  SELECT cast('t' as boolean); -- true (case insensitive)
  SELECT cast('true' as boolean); -- true (case insensitive)
  SELECT cast('f' as boolean); -- false (case insensitive)
  SELECT cast('false' as boolean); -- false (case insensitive)
  SELECT cast('F' as boolean); -- false (case insensitive)
  SELECT cast('T' as boolean); -- true (case insensitive)

Invalid examples

::

  SELECT cast('1.7E308' as boolean); -- Invalid argument
  SELECT cast('nan' as boolean); -- Invalid argument
  SELECT cast('infinity' as boolean); -- Invalid argument
  SELECT cast('12' as boolean); -- Invalid argument
  SELECT cast('-1' as boolean); -- Invalid argument
  SELECT cast('tr' as boolean); -- Invalid argument
  SELECT cast('tru' as boolean); -- Invalid argument
  SELECT cast('No' as boolean); -- Invalid argument

Cast to Floating-Point Types
----------------------------

From integral or floating-point types
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Casting from an integral or floating-point number is allowed.

Valid examples

::

  SELECT cast(1 as real); -- 1.0
  SELECT cast(123.45 as real); -- 123.45

There are two cases where Velox behaves differently from Presto (:issue:`5934`) when casting
to real from a value beyond real's limit. We will fix them to follow Presto's
behavior.

::

  SELECT cast(1.7E308 as real); -- Presto returns Infinity but Velox throws
  SELECT cast(-1.7E308 as real); -- Presto returns -Infinity but Velox throws

From VARCHAR
^^^^^^^^^^^^

Casting a string to real is allowed if the string represents an integral or
floating-point number. Casting from invalid input values throws.

Valid examples

::

  SELECT cast('1.' as real); -- 1.0
  SELECT cast('1' as real); -- 1.0
  SELECT cast('1.7E308' as real); -- Infinity
  SELECT cast('Infinity' as real); -- Infinity (case sensitive)
  SELECT cast('-Infinity' as real); -- -Infinity (case sensitive)
  SELECT cast('NaN' as real); -- NaN (case sensitive)

Invalid examples

::

  SELECT cast('1.2a' as real); -- Invalid argument
  SELECT cast('1.2.3' as real); -- Invalid argument
  SELECT cast('infinity' as real); -- Invalid argument
  SELECT cast('-infinity' as real); -- -Invalid argument
  SELECT cast('inf' as real); -- Invalid argument
  SELECT cast('InfiNiTy' as real); -- Invalid argument
  SELECT cast('INFINITY' as real); -- Invalid argument
  SELECT cast('nAn' as real); -- Invalid argument
  SELECT cast('nan' as real); -- Invalid argument

Below cases are supported in Presto, but throw in Velox.

::

  SELECT cast('1.2f' as real); -- 1.2
  SELECT cast('1.2f' as double); -- 1.2
  SELECT cast('1.2d' as real); -- 1.2
  SELECT cast('1.2d' as double); -- 1.2

From decimal
^^^^^^^^^^^^

Casting from decimal to double, float or any integral type is allowed. During decimal to an integral type conversion, if result overflows, or underflows, an exception is thrown.

Valid example

::

  SELECT cast(decimal '10.001' as double); -- 10.001

Invalid example

::

  SELECT cast(decimal '300.001' as tinyint); -- Out of range

Cast to VARCHAR
---------------

Casting from scalar types to string is allowed.

Valid examples

::

  SELECT cast(123 as varchar); -- '123'
  SELECT cast(123.45 as varchar); -- '123.45'
  SELECT cast(123.0 as varchar); -- '123.0'
  SELECT cast(nan() as varchar); -- 'NaN'
  SELECT cast(infinity() as varchar); -- 'Infinity'
  SELECT cast(true as varchar); -- 'true'
  SELECT cast(timestamp '1970-01-01 00:00:00' as varchar); -- '1970-01-01 00:00:00.000'
  SELECT cast(timestamp '2024-06-01 11:37:15.123 America/New_York' as varchar); -- '2024-06-01 11:37:15.123 America/New_York'
  SELECT cast(cast(22.51 as DECIMAL(5, 3)) as varchar); -- '22.510'
  SELECT cast(cast(-22.51 as DECIMAL(4, 2)) as varchar); -- '-22.51'
  SELECT cast(cast(0.123 as DECIMAL(3, 3)) as varchar); -- '0.123'
  SELECT cast(cast(1 as DECIMAL(6, 2)) as varchar); -- '1.00'
  SELECT cast(cast(0 as DECIMAL(6, 2)) as varchar); -- '0.00'

From Floating-Point Types
^^^^^^^^^^^^^^^^^^^^^^^^^

By default, casting a real or double to string returns standard notation if the magnitude of input value is greater than
or equal to 10 :superscript:`-3` but less than 10 :superscript:`7`, and returns scientific notation otherwise.

Positive zero returns '0.0' and negative zero returns '-0.0'. Positive infinity returns 'Infinity' and negative infinity
returns '-Infinity'. Positive and negative NaN returns 'NaN'.

If legacy_cast configuration property is true, the result is standard notation for all input value.

Valid examples if legacy_cast = false,

::

  SELECT cast(double '123456789.01234567' as varchar); -- '1.2345678901234567E8'
  SELECT cast(double '10000000.0' as varchar); -- '1.0E7'
  SELECT cast(double '12345.0' as varchar); -- '12345.0'
  SELECT cast(double '-0.001' as varchar); -- '-0.001'
  SELECT cast(double '-0.00012' as varchar); -- '-1.2E-4'
  SELECT cast(double '0.0' as varchar); -- '0.0'
  SELECT cast(double '-0.0' as varchar); -- '-0.0'
  SELECT cast(infinity() as varchar); -- 'Infinity'
  SELECT cast(-infinity() as varchar); -- '-Infinity'
  SELECT cast(nan() as varchar); -- 'NaN'
  SELECT cast(-nan() as varchar); -- 'NaN'

  SELECT cast(real '123456780.0' as varchar); -- '1.2345678E8'
  SELECT cast(real '10000000.0' as varchar); -- '1.0E7'
  SELECT cast(real '12345.0' as varchar); -- '12345.0'
  SELECT cast(real '-0.001' as varchar); -- '-0.001'
  SELECT cast(real '-0.00012' as varchar); -- '-1.2E-4'
  SELECT cast(real '0.0' as varchar); -- '0.0'
  SELECT cast(real '-0.0' as varchar); -- '-0.0'

Valid examples if legacy_cast = true,

::

  SELECT cast(double '123456789.01234567' as varchar); -- '123456789.01234567'
  SELECT cast(double '10000000.0' as varchar); -- '10000000.0'
  SELECT cast(double '-0.001' as varchar); -- '-0.001'
  SELECT cast(double '-0.00012' as varchar); -- '-0.00012'

  SELECT cast(real '123456780.0' as varchar); -- '123456784.0'
  SELECT cast(real '10000000.0' as varchar); -- '10000000.0'
  SELECT cast(real '12345.0' as varchar); -- '12345.0'
  SELECT cast(real '-0.00012' as varchar); -- '-0.00011999999696854502'


From DATE
^^^^^^^^^

Casting DATE to VARCHAR returns an ISO-8601 formatted string: YYYY-MM-DD.

::

    SELECT cast(date('2024-03-14') as varchar); -- '2024-03-14'


From TIMESTAMP
^^^^^^^^^^^^^^

By default, casting a timestamp to a string returns ISO 8601 format with space as separator
between date and time, and the year part is padded with zeros to 4 characters.

If legacy_cast configuration property is true, the result string uses character 'T'
as separator between date and time and the year part is not padded.

Valid examples if legacy_cast = false,

::

  SELECT cast(timestamp '1970-01-01 00:00:00' as varchar); -- '1970-01-01 00:00:00.000'
  SELECT cast(timestamp '2000-01-01 12:21:56.129' as varchar); -- '2000-01-01 12:21:56.129'
  SELECT cast(timestamp '384-01-01 08:00:00.000' as varchar); -- '0384-01-01 08:00:00.000'
  SELECT cast(timestamp '10000-02-01 16:00:00.000' as varchar); -- '10000-02-01 16:00:00.000'
  SELECT cast(timestamp '-10-02-01 10:00:00.000' as varchar); -- '-0010-02-01 10:00:00.000'

Valid examples if legacy_cast = true,

::

  SELECT cast(timestamp '1970-01-01 00:00:00' as varchar); -- '1970-01-01T00:00:00.000'
  SELECT cast(timestamp '2000-01-01 12:21:56.129' as varchar); -- '2000-01-01T12:21:56.129'
  SELECT cast(timestamp '384-01-01 08:00:00.000' as varchar); -- '384-01-01T08:00:00.000'
  SELECT cast(timestamp '-10-02-01 10:00:00.000' as varchar); -- '-10-02-01T10:00:00.000'

From INTERVAL DAY TO SECOND
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Casting INTERVAL DAY TO SECOND to VARCHAR returns a string formatted as
'[sign]D HH:MM:SS.ZZZ', where 'sign' is an optional '-' sign if interval is negative, D
is the number of whole days in the interval, HH is then number of hours between 00 and
24, MM is the number of minutes between 00 and 59, SS is the number of seconds between
00 and 59, and zzz is the number of milliseconds between 000 and 999.

::

    SELECT cast(interval '1' day as varchar); -- '1 00:00:00.000'
    SELECT cast(interval '123456' second as varchar); -- '1 10:17:36.000'
    SELECT cast(now() - date('2024-03-01') as varchar); -- '35 09:15:54.092'
    SELECT cast(date('2024-03-01') - now() as varchar); -- '-35 09:16:20.598'

From IPADDRESS
^^^^^^^^^^^^^^

Casting from IPADDRESS to VARCHAR returns a string formatted as x.x.x.x for IPV4 formatted IPV6 addresses.
For all other IPV6 addresses it will be formatted in compressed alternate form IPV6 defined in `RFC 4291#section-2.2 <https://datatracker.ietf.org/doc/html/rfc4291.html#section-2.2>`_

IPV4:

::

  SELECT cast(ipaddress '1.2.3.4' as varchar); -- '1.2.3.4'

IPV6:

::

  SELECT cast(ipaddress '2001:0db8:0000:0000:0000:ff00:0042:8329' as varchar); -- '2001:db8::ff00:42:8329'
  SELECT cast(ipaddress '0:0:0:0:0:0:13.1.68.3' as varchar); -- '::13.1.68.3'

IPV4 mapped IPV6:

::

  SELECT cast(ipaddress '::ffff:ffff:ffff' as varchar); -- '255.255.255.255'

From IPPREFIX
^^^^^^^^^^^^^

Casting from IPPREFIX to VARCHAR returns a string formatted as *x.x.x.x/<prefix-length>* for IPv4 formatted IPv6 addresses.

For all other IPv6 addresses it will be formatted in compressed alternate form IPv6 defined in `RFC 4291#section-2.2 <https://datatracker.ietf.org/doc/html/rfc4291.html#section-2.2>`_
followed by */<prefix-length>*. [`RFC 4291#section-2.3 <https://datatracker.ietf.org/doc/html/rfc4291.html#section-2.3>`_]

IPv4:

::

  SELECT cast(ipprefix '1.2.0.0/16' as varchar); -- '1.2.0.0/16'

IPv6:

::

  SELECT cast(ipprefix '2001:db8::ff00:42:8329/128' as varchar); -- '2001:db8::ff00:42:8329/128'
  SELECT cast(ipprefix '0:0:0:0:0:0:13.1.68.3/32' as varchar); -- '::/32'

IPv4 mapped IPv6:

::

  SELECT cast(ipaddress '::ffff:ffff:0000/16' as varchar); -- '255.255.0.0/16'

Cast to VARBINARY
-----------------

From IPADDRESS
^^^^^^^^^^^^^^

Returns the IPV6 address as a 16 byte varbinary string in network byte order.

Internally, the type is a pure IPv6 address. Support for IPv4 is handled using the IPv4-mapped IPv6 address range `(RFC 4291#section-2.5.5.2) <https://datatracker.ietf.org/doc/html/rfc4291.html#section-2.5.5.2>`_.
When creating an IPADDRESS, IPv4 addresses will be mapped into that range.

IPV6:

::

  SELECT cast(ipaddress '2001:0db8:0000:0000:0000:ff00:0042:8329' as varbinary); -- 0x20010db8000000000000ff0000428329

IPV4:

::

  SELECT cast('1.2.3.4' as ipaddress); -- 0x00000000000000000000ffff01020304

IPV4 mapped IPV6:

::

  SELECT cast('::ffff:ffff:ffff' as ipaddress); -- 0x00000000000000000000ffffffffffff

From TDIGEST(DOUBLE)
^^^^^^^^^^^^^^^^^^^^

Returns the T-digest as a varbinary string containing the serialized representation of the T-digest data structure.
This allows T-digests to be stored and retrieved for later use.

::

  SELECT cast(tdigest_agg(cast(1.0 as double)) as varbinary); -- AQAAAAAAAADwPwAAAAAAAPA/AAAAAAAA8D8AAAAAAABZQAAAAAAAAPA/AQAAAAAAAAAAAPA/AAAAAAAA8D8=

From QDIGEST(BIGINT), QDIGEST(REAL), QDIGEST(DOUBLE)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Returns the quantile digest as a varbinary string containing the serialized representation of the quantile digest data structure.
This allows quantile digests to be stored and retrieved for later use.

::

  SELECT cast(qdigest_agg(cast(1.0 as double)) as varbinary); -- AHsUrkfheoQ/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAPA/AAAAAAAA8D8BAAAAAAAAAAAAAPA/AAAAAAAA8L8=

Cast to TIMESTAMP
-----------------

From VARCHAR
^^^^^^^^^^^^

Casting from a string to timestamp is allowed if the string represents a
timestamp in the format `YYYY-MM-DD` followed by an optional `hh:mm:ss.MS`.
Seconds and milliseconds are optional. Casting from invalid input values throws.

Valid examples:

::

  SELECT cast('1970-01-01' as timestamp); -- 1970-01-01 00:00:00
  SELECT cast('1970-01-01 00:00:00.123' as timestamp); -- 1970-01-01 00:00:00.123
  SELECT cast('1970-01-01 02:01' as timestamp); -- 1970-01-01 02:01:00
  SELECT cast('1970-01-01 00:00:00-02:00' as timestamp); -- 1970-01-01 02:00:00

Invalid example:

::

  SELECT cast('2012-Oct-23' as timestamp); -- Invalid argument

Optionally, strings may also contain timezone information at the end. Timezone
information may be offsets in the format `+01:00` or `-02:00`, for example, or
timezone names, like `UTC`, `Z`, `America/Los_Angeles` and others,
`as defined here <https://github.com/facebookincubator/velox/blob/main/velox/type/tz/TimeZoneDatabase.cpp>`_.

For example, these strings contain valid timezone information:

::

  SELECT cast('1970-01-01 00:00:00 +09:00' as timestamp);
  SELECT cast('1970-01-01 00:00:00 UTC' as timestamp);
  SELECT cast('1970-01-01 00:00:00 America/Sao_Paulo' as timestamp);

If timezone information is specified in the string, the returned timestamp
is adjusted to the corresponding timezone. Otherwise, the timestamp is
assumed to be in the client session timezone, and adjusted accordingly
based on the value of `adjust_timestamp_to_session_timezone`, as described below.

The space between the hour and timezone definition is optional.

::

  SELECT cast('1970-01-01 00:00 Z' as timestamp);
  SELECT cast('1970-01-01 00:00Z' as timestamp);

Are both valid.

From DATE
^^^^^^^^^

Casting from date to timestamp is allowed.

Valid examples

::

  SELECT cast(date '1970-01-01' as timestamp); -- 1970-01-01 00:00:00
  SELECT cast(date '2012-03-09' as timestamp); -- 2012-03-09 00:00:00

From TIMESTAMP WITH TIME ZONE
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The results depend on whether configuration property `adjust_timestamp_to_session_timezone` is set or not.

If set to true, input timezone is ignored and timestamp is returned as is. For example,
"1970-01-01 00:00:00.000 America/Los_Angeles" becomes "1970-01-01 08:00:00.000".

Otherwise, timestamp is shifted by the offset of the timezone. For example,
"1970-01-01 00:00:00.000 America/Los_Angeles" becomes "1970-01-01 00:00:00.000".

Valid examples

::

  -- `adjust_timestamp_to_session_timezone` is true
  SELECT to_unixtime(cast(timestamp '1970-01-01 00:00:00 America/Los_Angeles' as timestamp)); -- 28800.0 (1970-01-01 08:00:00.000)
  SELECT to_unixtime(cast(timestamp '2012-03-09 10:00:00 Asia/Chongqing' as timestamp)); -- 1.3312584E9 (2012-03-09 02:00:00.000)
  SELECT to_unixtime(cast(from_unixtime(0, '+06:00') as timestamp)); -- 0.0 (1970-01-01 00:00:00.000)
  SELECT to_unixtime(cast(from_unixtime(0, '-02:00') as timestamp)); -- 0.0 (1970-01-01 00:00:00.000)

  -- `adjust_timestamp_to_session_timezone` is false
  SELECT to_unixtime(cast(timestamp '1970-01-01 00:00:00 America/Los_Angeles' as timestamp)); -- 0.0 (1970-01-01 00:00:00.000)
  SELECT to_unixtime(cast(timestamp '2012-03-09 10:00:00 Asia/Chongqing' as timestamp)); -- 1.3312872E9 (2012-03-09 10:00:00.000)
  SELECT to_unixtime(cast(from_unixtime(0, '+06:00') as timestamp)); -- 21600.0 (1970-01-01 06:00:00.000)
  SELECT to_unixtime(cast(from_unixtime(0, '-02:00') as timestamp)); -- -7200.0 (1969-12-31 22:00:00.000)

Cast to TIMESTAMP WITH TIME ZONE
--------------------------------

From TIMESTAMP
^^^^^^^^^^^^^^

The results depend on whether configuration property `adjust_timestamp_to_session_timezone` is set or not.

If set to true, the output is adjusted to be equivalent as the input timestamp in UTC
based on the user provided `session_timezone` (if any). For example, when user supplies
"America/Los_Angeles" "1970-01-01 00:00:00.000" becomes "1969-12-31 16:00:00.000 America/Los_Angeles".

Otherwise, the user provided `session_timezone` (if any) is simply appended to the input
timestamp. For example, "1970-01-01 00:00:00.000" becomes "1970-01-01 00:00:00.000 America/Los_Angeles".

Valid examples

::

  -- `adjust_timestamp_to_session_timezone` is true
  SELECT cast(timestamp '1970-01-01 00:00:00' as timestamp with time zone); -- 1969-12-31 16:00:00.000 America/Los_Angeles
  SELECT cast(timestamp '2012-03-09 10:00:00' as timestamp with time zone); -- 2012-03-09 02:00:00.000 America/Los_Angeles
  SELECT cast(from_unixtime(0) as timestamp with time zone); -- 1969-12-31 16:00:00.000 America/Los_Angeles

  -- `adjust_timestamp_to_session_timezone` is false
  SELECT cast(timestamp '1970-01-01 00:00:00' as timestamp with time zone); -- 1970-01-01 00:00:00.000 America/Los_Angeles
  SELECT cast(timestamp '2012-03-09 10:00:00' as timestamp with time zone); -- 2012-03-09 10:00:00.000 America/Los_Angeles
  SELECT cast(from_unixtime(0) as timestamp with time zone); -- 1970-01-01 00:00:00.000 America/Los_Angeles

From DATE
^^^^^^^^^

The results depend on `session_timestamp`.

Valid examples

::

    -- session_timezone = America/Los_Angeles
    SELECT cast(date '2024-06-01' as timestamp with time zone); -- 2024-06-01 00:00:00.000 America/Los_Angeles

    -- session_timezone = Asia/Shanghai
    SELECT cast(date '2024-06-01' as timestamp with time zone); -- 2024-06-01 00:00:00.000 Asia/Shanghai

Cast to Date
------------

From VARCHAR
^^^^^^^^^^^^

Only ISO 8601 strings are supported: `[+-]YYYY-MM-DD`. Casting from invalid input values throws.

Valid examples

::

  SELECT cast('1970-01-01' as date); -- 1970-01-01

Invalid examples

::

  SELECT cast('2012' as date); -- Invalid argument
  SELECT cast('2012-10' as date); -- Invalid argument
  SELECT cast('2012-10-23T123' as date); -- Invalid argument
  SELECT cast('2012-10-23 (BC)' as date); -- Invalid argument
  SELECT cast('2012-Oct-23' as date); -- Invalid argument
  SELECT cast('2012/10/23' as date); -- Invalid argument
  SELECT cast('2012.10.23' as date); -- Invalid argument
  SELECT cast('2012-10-23 ' as date); -- Invalid argument

From TIMESTAMP
^^^^^^^^^^^^^^

Casting from timestamp to date is allowed. If present, the part of `hh:mm:ss`
in the input is ignored.

Valid examples

::

  SELECT cast(timestamp '1970-01-01 00:00:00' as date); -- 1970-01-01
  SELECT cast(timestamp '1970-01-01 23:59:59' as date); -- 1970-01-01

From TIMESTAMP WITH TIME ZONE
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Casting from TIMESTAMP WITH TIME ZONE to DATE is allowed. If present,
the part of `hh:mm:ss` in the input is ignored.

Session time zone does not affect the result.

Valid examples

::

  SELECT CAST(timestamp '2024-06-01 01:38:00 America/New_York' as DATE); -- 2024-06-01

Cast to Decimal
---------------

From boolean type
^^^^^^^^^^^^^^^^^

Casting a boolean number to decimal of given precision and scale is allowed.
True value is converted to 1 and false to 0.

Valid examples

::

  SELECT cast(true as decimal(4, 2)); -- decimal '1.00'
  SELECT cast(false as decimal(8, 2)); -- decimal '0'

From integral types
^^^^^^^^^^^^^^^^^^^

Casting an integral number to a decimal of given precision and scale is allowed
if the input value can be represented by the precision and scale. Casting from
invalid input values throws.

Valid examples

::

  SELECT cast(1 as decimal(4, 2)); -- decimal '1.00'
  SELECT cast(10 as decimal(4, 2)); -- decimal '10.00'
  SELECT cast(123 as decimal(5, 2)); -- decimal '123.00'

Invalid examples

::

  SELECT cast(123 as decimal(6, 4)); -- Out of range
  SELECT cast(123 as decimal(4, 2)); -- Out of range

From floating-point types
^^^^^^^^^^^^^^^^^^^^^^^^^

Casting a floating-point number to a decimal of given precision and scale is allowed
if the input value can be represented by the precision and scale. When the given
scale is less than the number of decimal places, the floating-point value is rounded.
The conversion precision is up to 15 for double and 6 for real according to the
significant decimal digits precision they provide. Casting from NaN or infinite value
throws.

Valid example

::

  SELECT cast(0.12 as decimal(4, 4)); -- decimal '0.1200'
  SELECT cast(0.12 as decimal(4, 1)); -- decimal '0.1'
  SELECT cast(0.19 as decimal(4, 1)); -- decimal '0.2'
  SELECT cast(0.123456789123123 as decimal(38, 18)); -- decimal '0.123456789123123000'
  SELECT cast(real '0.123456' as decimal(38, 18)); -- decimal '0.123456000000000000'

Invalid example

::

  SELECT cast(123.12 as decimal(6, 4)); -- Out of range
  SELECT cast(99999.99 as decimal(6, 2)); -- Out of range

From decimal
^^^^^^^^^^^^

Casting one decimal to another is allowed if the input value can be represented
by the result decimal type. When casting from a larger scale to a smaller one,
the fraction part is rounded.

Valid example

::

  SELECT cast(decimal '0.69' as decimal(4, 3)); -- decimal '0.690'
  SELECT cast(decimal '0.69' as decimal(4, 1)); -- decimal '0.7'

Invalid example

::

  SELECT cast(decimal '-1000.000' as decimal(6, 4)); -- Out of range
  SELECT cast(decimal '123456789' as decimal(9, 1)); -- Out of range

From varchar
^^^^^^^^^^^^

Casting varchar to a decimal of given precision and scale is allowed
if the input value can be represented by the precision and scale. When casting from
a larger scale to a smaller one, the fraction part is rounded. Casting from invalid input value throws.

Valid example

::

  SELECT cast('9999999999.99' as decimal(12, 2)); -- decimal '9999999999.99'
  SELECT cast('1.556' as decimal(12, 2)); -- decimal '1.56'
  SELECT cast('1.554' as decimal(12, 2)); -- decimal '1.55'
  SELECT cast('-1.554' as decimal(12, 2)); -- decimal '-1.55'
  SELECT cast('+09' as decimal(12, 2)); -- decimal '9.00'
  SELECT cast('9.' as decimal(12, 2)); -- decimal '9.00'
  SELECT cast('.9' as decimal(12, 2)); -- decimal '0.90'
  SELECT cast('3E+2' as decimal(12, 2)); -- decimal '300.00'
  SELECT cast('3E+00002' as decimal(12, 2)); -- decimal '300.00'
  SELECT cast('3e+2' as decimal(12, 2)); -- decimal '300.00'
  SELECT cast('31.423e+2' as decimal(12, 2)); -- decimal '3142.30'
  SELECT cast('1.2e-2' as decimal(12, 2)); -- decimal '0.01'
  SELECT cast('1.2e-5' as decimal(12, 2)); -- decimal '0.00'
  SELECT cast('0000.123' as decimal(12, 2)); -- decimal '0.12'
  SELECT cast('.123000000' as decimal(12, 2)); -- decimal '0.12'

Invalid example

::

  SELECT cast('1.23e67' as decimal(38, 0)); -- Value too large
  SELECT cast('0.0446a' as decimal(9, 1)); -- Value is not a number
  SELECT cast('' as decimal(9, 1)); -- Value is not a number
  SELECT cast('23e-5d' as decimal(9, 1)); -- Value is not a number
  SELECT cast('1.23 ' as decimal(38, 0)); -- Value is not a number
  SELECT cast(' -3E+2' as decimal(12, 2)); -- Value is not a number
  SELECT cast('-3E+2.1' as decimal(12, 2)); -- Value is not a number
  SELECT cast('3E+' as decimal(12, 2)); -- Value is not a number

Cast to IPADDRESS
-----------------

.. _ipaddress-from-varchar:

From VARCHAR
^^^^^^^^^^^^

To cast a varchar to IPAddress input string must be in the form of either
IPV4 or IPV6.

For IPV4 it must be in the form of:
x.x.x.x where each x is an integer value between 0-255.

For IPV6 it must follow any of the forms defined in `RFC 4291#section-2.2 <https://datatracker.ietf.org/doc/html/rfc4291.html#section-2.2>`_.

Full form:

::

   2001:0DB8:0000:0000:0008:0800:200C:417A
   2001:DB8:0:0:8:800:200C:417A

Compressed form:
::

   2001:DB8::8:800:200C:417A

Alternate form:
::

   0:0:0:0:0:0:13.1.68.3
   ::13.1.68.3

Internally, the type is a pure IPv6 address. Support for IPv4 is handled using the IPv4-mapped IPv6 address range `(RFC 4291#section-2.5.5.2) <https://datatracker.ietf.org/doc/html/rfc4291.html#section-2.5.5.2>`_.
When creating an IPADDRESS, IPv4 addresses will be mapped into that range.

When formatting an IPADDRESS, any address within the mapped range will be formatted as an IPv4 address.
Other addresses will be formatted as IPv6 using the canonical format defined in `RFC 5952 <https://datatracker.ietf.org/doc/html/rfc5952.html>`_.

Valid examples:

::

  SELECT cast('2001:0db8:0000:0000:0000:ff00:0042:8329' as ipaddress); -- ipaddress '2001:db8::ff00:42:8329'
  SELECT cast('1.2.3.4' as ipaddress); -- ipaddress '1.2.3.4'
  SELECT cast('::ffff:ffff:ffff' as ipaddress); -- ipaddress '255.255.255.255'

Invalid examples:

::

  SELECT cast('2001:db8::1::1' as ipaddress); -- Invalid IP address '2001:db8::1::1'
  SELECT cast('789.1.1.1' as ipaddress); -- Invalid IP address '789.1.1.1'

From VARBINARY
^^^^^^^^^^^^^^

To cast a varbinary to IPAddress it must be either IPV4(4 Bytes)
or IPV6(16 Bytes) in network byte order.

IPV4:

::

[01, 02, 03, 04] -> 1.2.3.4

IPV6:

::

[0x20, 0x01, 0x0d, 0xb8 0x00, 0x00, 0x00, 0x00 0x00 0x00, 0xff, 0x00, 0x00, 0x42, 0x83, 0x29] -> 2001:db8::ff00:42:8329

Internally, the type is a pure IPv6 address. Support for IPv4 is handled using the IPv4-mapped IPv6 address range `(RFC 4291#section-2.5.5.2) <https://datatracker.ietf.org/doc/html/rfc4291.html#section-2.5.5.2>`_.
When creating an IPADDRESS, IPv4 addresses will be mapped into that range.

When formatting an IPADDRESS, any address within the mapped range will be formatted as an IPv4 address.
Other addresses will be formatted as IPv6 using the canonical format defined in `RFC 5952 <https://datatracker.ietf.org/doc/html/rfc5952.html>`_.

IPV6 mapped IPV4 address:

::

[0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0xff, 0xff, 0x01, 0x02, 0x03, 0x04] -> 1.2.3.4

Valid examples:

::

  SELECT cast(from_hex('20010db8000000000000ff0000428329') as ipaddress); -- ipaddress '2001:db8::ff00:42:8329'
  SELECT cast(from_hex('01020304') as ipaddress); -- ipaddress '1.2.3.4'
  SELECT cast(from_hex('00000000000000000000ffff01020304') as ipaddress); -- ipaddress '1.2.3.4'

Invalid examples:

::

  SELECT cast(from_hex('f000001100') as ipaddress); -- Invalid IP address binary length: 5

From IPPREFIX
^^^^^^^^^^^^^

Returns the canonical(lowest) IPADDRESS in the subnet range.

Examples:

::

  SELECT cast(ipprefix '1.2.3.4/24' as ipaddress) -- ipaddress '1.2.3.0'
  SELECT cast(ipprefix '2001:db8::ff00:42:8329/64' as ipaddress) -- ipaddress '2001:db8::'

Cast to TDIGEST(DOUBLE)
-----------------------

From VARBINARY
^^^^^^^^^^^^^^

Returns a T-digest reconstructed from the varbinary string containing the serialized representation.
This allows previously stored T-digests to be restored for use.

::

  SELECT cast(stored_tdigest_binary as tdigest(double));

Cast to QDIGEST(BIGINT), QDIGEST(REAL), QDIGEST(DOUBLE)
-------------------------------------------------------

From VARBINARY
^^^^^^^^^^^^^^

Returns a quantile digest reconstructed from the varbinary string containing the serialized representation.
This allows previously stored quantile digests to be restored for use.

::

  SELECT cast(stored_qdigest_binary as qdigest(bigint));
  SELECT cast(stored_qdigest_binary as qdigest(real));
  SELECT cast(stored_qdigest_binary as qdigest(double));

Cast to IPPREFIX
----------------

From VARCHAR
^^^^^^^^^^^^

The IPPREFIX string must be in the form of *<ip_address>/<ip_prefix>* as defined in `RFC 4291#section-2.3 <https://datatracker.ietf.org/doc/html/rfc4291.html#section-2.3>`_.
The IPADDRESS portion of the IPPREFIX follows the same rules as casting
`IPADDRESS from VARCHAR <#ipaddress-from-varchar>`_.

The prefix portion must be <= 32 if the IP is an IPv4 address or <= 128 for an IPv6 address.
As with IPADDRESS, any IPv6 address in the form of an IPv4 mapped IPv6 address will be
interpreted as an IPv4 address. Only the canonical(smallest) IP address will be stored
in the IPPREFIX.

Examples:

Valid examples:

::

  SELECT cast('2001:0db8:0000:0000:0000:ff00:0042:8329/32' as ipprefix); -- ipprefix '2001:0db8::/32'
  SELECT cast('1.2.3.4/24' as ipprefix); -- ipprefix '1.2.3.0/24'
  SELECT cast('::ffff:ffff:ffff/16' as ipprefix); -- ipprefix '255.255.0.0/16'

Invalid examples:

::

  SELECT cast('2001:db8::1::1/1' as ipprefix); -- Cannot cast value to IPPREFIX: 2001:db8::1::1/1
  SELECT cast('2001:0db8:0000:0000:0000:ff00:0042:8329/129' as ipprefix); -- Cannot cast value to IPPREFIX: 2001:0db8:0000:0000:0000:ff00:0042:8329/129
  SELECT cast('2001:0db8:0000:0000:0000:ff00:0042:8329/-1' as ipprefix); -- Cannot cast value to IPPREFIX: 2001:0db8:0000:0000:0000:ff00:0042:8329/-1
  SELECT cast('255.2.3.4/33' as ipprefix); -- Cannot cast value to IPPREFIX: 255.2.3.4/33
  SELECT cast('::ffff:ffff:ffff/33' as ipprefix); -- Cannot cast value to IPPREFIX: ::ffff:ffff:ffff/33

From IPADDRESS
^^^^^^^^^^^^^^

Returns an IPPREFIX where the prefix length is the length of the entire IP address.
Prefix length for IPv4 is 32 and for IPv6 it is 128.

Examples:

::

  SELECT cast(ipaddress '1.2.3.4' as ipprefix) -- ipprefix '1.2.3.4/32'
  SELECT cast(ipaddress '2001:db8::ff00:42:8329' as ipprefix) -- ipprefix '2001:db8::ff00:42:8329/128'

Data Size Functions
-------------------

.. function:: parse_presto_data_size(string) -> decimal(38)

    Parses ``string`` of format ``value unit`` into a number, where ``value`` is the fractional number of unit values::

      SELECT parse_presto_data_size('1B'); -- 1
      SELECT parse_presto_data_size('1kB'); -- 1024
      SELECT parse_presto_data_size('1MB'); -- 1048576
      SELECT parse_presto_data_size('2.3MB'); -- 2411724

Miscellaneous
-------------

.. function:: typeof(x) -> varchar

    Returns the name of the type of x::

        SELECT typeof(123); -- integer
        SELECT typeof(1.5); -- double
        SELECT typeof(array[1,2,3]); -- array(integer)
