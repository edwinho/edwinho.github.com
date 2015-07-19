---
author: edwin
comments: true
date: 2012-10-10 15:54:36+00:00
layout: post
slug: the-fundamentals-of-python
title: "The fundamentals of Python"
wordpress_id: 93
categories:
- codes
tags:
- Python
---

Here are the notes of my Python(basic) self-study.


**Comments**




single line comments




\# balabala




Multi-Line comments




""" balabala




balabala"""










**Math Operations**








There are six arithmetic operators we're going to focus on:



	
  1. Addition (`+`)

	
  2. Subtraction (`-`)

	
  3. Multiplication (`*`)

	
  4. Division (`/`)

	
  5. Exponentiation (`**`)

	
  6. Modulo (`%`)





**Strings**




'bala'  or  "bala"




if ' in the string,The backslash character (`\`) does this work for us! Like this: " I\'m a boy."






We're going to focus on four string methods in this section:



	
  1. `len()    :len(string)`

	
  2. `lower()  :string.lower()`

	
  3. `upper()  :string.upper()`

	
  4. `str()    :str(bala)`







we also can use String like this: "bala" + "balabala"







_String Formatting with %, part I_






camelot = "Camelot"  
place = "place"  
print "Let's not go to %s. 'Tis a silly %s." % (camelot, place)









_String Formatting with %, part II_






name = raw_input("What is your name?")  
quest = raw_input("What is your quest?")  
color = raw_input("What is your favorite color?")

print "Ah, so your name is %s, your quest is %s, \  
and your favorite color is %s." % (name, quest, color)









_check that the word is also composed of all alphabetical characters._




string.isalpha()









 If you have a string `s`, you can get the "slice" of `s` from `i` to `j` using`s[i:j]`. This gives you the characters from position `i` to `j`.



	
  * For example, if `s = "foo"`, then`s[0:2]` gives you `"fo"`.








**Boolean Operations**

There are three boolean operators in Python:



	
  1. `and`, which means the same as it does in English;

	
  2. `or`, which means "one or the other OR BOTH" (it's not _exclusively_ one or the other, the way it often is in English);

	
  3. `not`, which means the same as it does in English.









there is an **order of precedence** or **order of operations** for boolean operators. The order is as follows:



	
  1. `not` is evaluated first;

	
  2. `and` is evaluated next;

	
  3. `or` is evaluated last.


This order can be changed by including parentheses(`()`). Anything in parentheses is evaluated as its own unit.




**Date and Time**




 `from datetime import datetime`. Importing special functionality into your programs.




_Getting the current date and time_:




datetime.now()




_Extracting Information_:




now.year,  now.hour   (month,day,minute,second,etc.)






