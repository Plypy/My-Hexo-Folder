title: VHDL notes type subtype and generate
date: 2014-11-08 11:14:33
tags:
- VHDL
categories:
- LerningNotes
---

Last week's Lab of Computer Organization Theory, we are asked to implement something that is constituted of several components, so called a hierarchical design, or rather, top down design.

Anyway, what we have to do is to code serveral basic blocks, and then connect them to form a high-level design.

## type and subtype

While coding, I found that, I was constantly repeating to write something like,

    a0: in std_logic_vector(15 downto 0);
    a1: in std_logic_vector(15 downto 0);
    a2: in std_logic_vector(15 downto 0);
    a3: in std_logic_vector(15 downto 0);

which is quiet ugly indeed. I was just repeating myself.

I want to spare my life, so I wonder if there is a 2-dimensional array of `std_logic`, then I can return to the happay days that I have in C, `std_logic a[4][15]`.

There is quiet a lot resources online, so without many pain, I found myself the `User defined types`. Since what I want is an array of `std_logic_vector`, I wrote these,

    -- alias for std_logic_vector(15 downto 0)
    subtype std_logic_16 is std_logic_vector(15 downto 0);
    -- Array type for std_logic_vector
    type std_logic_ar16 is array(natural range <>) of std_logic_16;

Now, I can write

    a: in std_logic_ar16(3 downto 0);

The life is wonderful again! However, the VHDL is different from high-level language, the array type is different from ordinary type. If you want to declare an array, you must use array type, the same goes for nonarray type. In detail, if you want to have a single instance of `std_logic_vector`, you must write `a: std_logic_16` , instead of `a: std_logic_ar16`. 

That seems to be evident, however, I once thought the `std_vector_array(15 downto 0)` as the `int [15]`, so I made a stupid mistake regarding `std_vector_array` as the `int`. anyway...


## for generate

While I was mapping the components, the same thing happened again. Say, I have to connect 16 and gates with 16 out ports of an unit. I need to write the `port map` thing for 16 times... 

The people created VHDL aren't fools, they have [for generate](http://www.ics.uci.edu/~jmoorkan/vhdlref/generate.html).

Life is once again wonderful.
