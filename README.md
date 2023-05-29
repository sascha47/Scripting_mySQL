# Scripting_mySQL

```

#!/usr/bin/python3
#2023/05/14
#check md5 hash of file in files table in cmdb and compare it to current file hash in cli; write result to table

import pymysql
import csv
import hashlib
import subprocess

monitor_csv = "monitor.csv" # file to write with

# file to write to
with open(monitor_csv,"w") as file:
    writer = csv.writer(file)
    writer.writerow(["filepath", "dbash", "db hash date", "current hash", "status"]) # columns

    db = pymysql.connect(host="127.0.0.1", # connection to the db
    user = "cmdb",
    password = "107043",
    database = "cmdb",
    port = 3306
    )

    cursor = db.cursor() # pointer object for the db
    

    cursor.execute("SELECT * FROM files") # select everything from file table

    result = cursor.fetchall() # return evertyhing from files table
    #print(result)

    for row in result: # loop through index of files tables and 
        filepath = row[0]
        time = row[1]
        md5_hash = row[2]
     # md5 checksu on file path from linux
        current = subprocess.run(f"md5sum {filepath} | awk '{{print $1}}'",shell=True,universal_newlines=True,capture_output=True,text=True)
        if current.stdout.strip() == md5_hash: # compare to md5_hash of file path from DB
            okay = "ok" # if same okay
            writer.writerow([filepath,md5_hash,time,current.stdout.strip(),okay]) # write to csv
        else:
            changed = "file changed" # if md5 hash changed say so
            writer.writerow([filepath,md5_hash,time,current.stdout.strip(),changed]) # write to file
 ```
