VHDL Type Conversion Using Numeric_std Library
===

Modified form the original post: https://www.nandland.com/vhdl/tips/tip-convert-numeric-std-logic-vector-to-integer.html

>Note that many of the below examples use the 'length VHDL attribute. This attribute makes your code more portable and versatile, so it should be used.

- Integer to Signed
- Integer to Std_Logic_Vector (<font color=red>no direct conversion</font>)
- Integer to Unsigned
- Std_Logic_Vector To Integer (<font color=red>no direct conversion</font>)
- Std_Logic_Vector To Signed
- Std_Logic_Vector To Unsigned
- Signed to Integer
- Signed to Std_Logic_Vector
- Signed to Unsigned
- Unsigned to Integer
- Unsigned to Signed
- Unsigned to Std_Logic_Vector

![](https://www.doulos.com/media/1486/numeric_std_conversions.gif)


# 1. Convert from Integer to Signed using Numeric_Std

The below example uses the to_signed conversion, which requires two input parameters. The first is the signal that you want to convert, the second is the length of the resulting vector.

```VHDL
signal input_3  : integer;
signal output_3 : signed(3 downto 0);
   
output_3 <= to_signed(input_3, output_3'length);
```

# 2. Convert from Integer to Std_Logic_Vector using Numeric_Std

First you need to think about the range of values stored in your integer. Can your integer be positive and negative? If so, you will need to use the to_signed() conversion. If your integer is only positive, you will need to use the to_unsigned() conversion.

Both of these conversion functions require two input parameters. The first is the signal that you want to convert, the second is the length of the resulting vector.

```VHDL
signal input_1   : integer;
signal output_1a : std_logic_vector(3 downto 0);
signal output_1b : std_logic_vector(3 downto 0);
   
-- This line demonstrates how to convert positive integers
output_1a <= std_logic_vector(to_unsigned(input_1, output_1a'length));
 
-- This line demonstrates how to convert positive or negative integers
output_1b <= std_logic_vector(to_signed(input_1, output_1b'length));
```

# 3. Convert from Integer to Unsigned using Numeric_Std

The below example uses the to_unsigned conversion, which requires two input parameters. The first is the signal that you want to convert, the second is the length of the resulting vector.

```VHDL
signal input_2  : integer;
signal output_2 : unsigned(3 downto 0);
   
output_2 <= to_unsigned(input_2, output_2'length);
```

# 4. Convert from Std_Logic_Vector to Integer using Numeric_Std

First you need to think about the data that is represented by your std_logic_vector. Is it signed data or is it unsigned data? Signed data means that your std_logic_vector can be a positive or negative number. Unsigned data means that your std_logic_vector is only a positive number. The example below uses the unsigned() typecast, but if your data can be negative you need to use the signed() typecast. Once you cast your input std_logic_vector as unsigned or signed, then you can convert it to integer as shown below:

```VHDL
signal input_4   : std_logic_vector(3 downto 0);
signal output_4a : integer;
signal output_4b : integer;
   
-- This line demonstrates the unsigned case
output_4a <= to_integer(unsigned(input_4));
 
-- This line demonstrates the signed case
output_4b <= to_integer(signed(input_4));
```

# 5. Convert from Std_Logic_Vector to Signed using Numeric_Std

This is an easy conversion, all you need to do is cast the std_logic_vector as signed as shown below:

```VHDL	
signal input_6  : std_logic_vector(3 downto 0);
signal output_6 : signed(3 downto 0);
 
output_6 <= signed(input_6);
```

# 6. Convert from Std_Logic_Vector to Unsigned using Numeric_Std

This is an easy conversion, all you need to do is cast the std_logic_vector as unsigned as shown below:

```VHDL	
signal input_5  : std_logic_vector(3 downto 0);
signal output_5 : unsigned(3 downto 0);
   
output_5 <= unsigned(input_5);
```

# 7. Convert from Signed to Integer using Numeric_Std

This is an easy conversion, all you need to do is use the to_integer function call from numeric_std as shown below:

```VHDL	
signal input_10  : signed(3 downto 0);
signal output_10 : integer;
 
output_10 <= to_integer(input_10);
```

# 8. Convert from Signed to Std_Logic_Vector using Numeric_Std

This is an easy conversion, all you need to do is use the std_logic_vector cast as shown below:

```VHDL	
signal input_11  : signed(3 downto 0);
signal output_11 : std_logic_vector(3 downto 0);
 
output_11 <= std_logic_vector(input_11);
```

# 9. Convert from Signed to Unsigned using Numeric_Std

This is an easy conversion, all you need to do is use the unsigned cast as shown below:

```VHDL	
signal input_12  : signed(3 downto 0);
signal output_12 : unsigned(3 downto 0);
   
output_12 <= unsigned(input_12);
```

# 10. Convert from Unsigned to Integer using Numeric_Std

This is an easy conversion, all you need to do is use the to_integer function call from numeric_std as shown below:

```	VHDL
signal input_7  : unsigned(3 downto 0);
signal output_7 : integer;
 
output_7 <= to_integer(input_7);
```

# 11. Convert from Unsigned to Signed using Numeric_Std

This is an easy conversion, all you need to do is use the signed cast as shown below:

```VHDL	
signal input_9  : unsigned(3 downto 0);
signal output_9 : signed(3 downto 0);
 
output_9 <= signed(input_9);
```

# 12. Convert from Unsigned to Std_Logic_Vector using Numeric_Std

This is an easy conversion, all you need to do is use the std_logic_vector cast as shown below:

```VHDL
signal input_8  : unsigned(3 downto 0);
signal output_8 : std_logic_vector(3 downto 0);
 
output_8 <= std_logic_vector(input_8);
```

