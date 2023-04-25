
https://www.cnblogs.com/lovelylife/p/5542363.html

在线测试： http://www.lua.org/cgi-bin/demo

````
function split( s, c )
    for item in string.gmatch( s, "(.-)"..c) do
        print(item);
    end
end

s = "12.12.12.12;122.1111.111.1;";
s2 = "00:00:00-04:00:00;00:00:00-07:00:00;"
s3 = "a,b,c,d,"

split( s, ";" );
split( s2, ":" );
split( s3, "," );
````

