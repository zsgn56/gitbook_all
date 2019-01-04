1

```py
#!/usr/bin/env python
#-*-coding:utf-8-*-
import time
time1 = time.time()
print(time1)
with open('1.txt') as file_object:
    lines_1 = file_object.readlines()
    file_line={}
    for line_1 in lines_1:
        line_1 = line_1.rstrip()        
        line_len1 = len(line_1)
        my_hash = 0

        for i in range(0,line_len1):
            my_hash = my_hash*33 + ord((line_1[i:i+1]))
            if my_hash < 0:
                my_hash = my_hash * (-1)

        file_line[my_hash]=line_1   


with open('2.txt') as file_object1:
    with open('result.txt', 'w') as file_object2:
        lines_2 = file_object1.readlines()
        for line_2 in lines_2:
            line_2 = line_2.rstrip()
            line_len2 = len(line_2)
            hash_value = 0

            for i in range(0, line_len2):
                hash_value = hash_value*33 + ord((line_2[i:i+1]))
                if hash_value < 0:
                    hash_value = hash_value * (-1)

            if hash_value in file_line.keys():
                result_line = file_line.get(hash_value) + '\n'
                file_object2.write(result_line)


time2 = time.time()
print(time2)

```



