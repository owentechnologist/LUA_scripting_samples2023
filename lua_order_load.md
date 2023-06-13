### The following calls to LUA will populate thousands of records --all in the same slot as directed by the value inside of the curly braces that is passed to the scripts as the KEYS[1] value to be used  
### Change the key argument to {2} and re-execute both of the EVAL commands that follow to write to a second slot

#### This first script creates a range of reusable values that are the sources for data generation in the script that later generates hashes:
#### Be certain to execute the script with varying values in the curly braces if you intend to have hashes live in different slots
#### {1} amd {2} should do nicely for most scenarios
```
EVAL "do redis.call('SADD','statuses'..KEYS[1],'cancelled','damaged','processing','placed','shipped','delivered') redis.call('SADD','productcats'..KEYS[1],'household','automotive','collectibles','crafts','decor','toys','bags','belts','wallets','hats','jewellry','scarves','sunglasses','lingerie','pants','skirts','cellphones','cameras','food','bath','medicine','vitamins','tools','supplies','furniture','gardening','kitchen','pets','rugs','linens','adult','orthotics','prosthetics','office','football','hunting','snowboarding','swimming','household') redis.call('SADD','charblocks'..KEYS[1],'abhoyzAOTVZ','WdefklmnVWX','YtrDhkiYcnB','LuGfeInUbvF','KlEEugRAAAA','lDDerSkGGFF','mndtRRewAAk','GGfsptrSrrS','EErfdjJvVVa','swqaNgfNMWW') redis.call('SADD','alpha'..KEYS[1],'a','b','c','d','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z','A','B','C','D','E','F','G','H','I','J','K','L','M','N','O','P','Q','R','S','T','U','V','W','X','Y','Z') redis.call('SADD','year'..KEYS[1],'2022','2021','2020','2019','2018','2017','2016','2015','2014','2013','2012','2011','2010','2009','2008','2007','2006','2005') redis.call('SADD','month'..KEYS[1],'01','02','03','04','05','06','07','08','09','10','11','12') redis.call('SADD','day'..KEYS[1],'1','2','3','4','5','6','7','8','9','10','11','12','13','14','15','16','17','18','19','20','21','22','23','24','25','26','27','28') redis.call('SADD','colors'..KEYS[1],'blue','green','purple','orange','yellow','red','silver','gold','white','black','grey') end return 'done'" 1 {1}
EVAL "do redis.call('SADD','statuses'..KEYS[1],'cancelled','damaged','processing','placed','shipped','delivered') redis.call('SADD','productcats'..KEYS[1],'household','automotive','collectibles','crafts','decor','toys','bags','belts','wallets','hats','jewellry','scarves','sunglasses','lingerie','pants','skirts','cellphones','cameras','food','bath','medicine','vitamins','tools','supplies','furniture','gardening','kitchen','pets','rugs','linens','adult','orthotics','prosthetics','office','football','hunting','snowboarding','swimming','household') redis.call('SADD','charblocks'..KEYS[1],'abhoyzAOTVZ','WdefklmnVWX','YtrDhkiYcnB','LuGfeInUbvF','KlEEugRAAAA','lDDerSkGGFF','mndtRRewAAk','GGfsptrSrrS','EErfdjJvVVa','swqaNgfNMWW') redis.call('SADD','alpha'..KEYS[1],'a','b','c','d','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z','A','B','C','D','E','F','G','H','I','J','K','L','M','N','O','P','Q','R','S','T','U','V','W','X','Y','Z') redis.call('SADD','year'..KEYS[1],'2022','2021','2020','2019','2018','2017','2016','2015','2014','2013','2012','2011','2010','2009','2008','2007','2006','2005') redis.call('SADD','month'..KEYS[1],'01','02','03','04','05','06','07','08','09','10','11','12') redis.call('SADD','day'..KEYS[1],'1','2','3','4','5','6','7','8','9','10','11','12','13','14','15','16','17','18','19','20','21','22','23','24','25','26','27','28') redis.call('SADD','colors'..KEYS[1],'blue','green','purple','orange','yellow','red','silver','gold','white','black','grey') end return 'done'" 1 {2}
```
#### The following LUA script uses the above SETS to generate Hashes: up to 250 thousand can be easily generated (after that the LUA can timeout)
#### to create more - adjust the arguments to start and finish at different index values (example: 250000 500000)

```
EVAL "for index=ARGV[2],ARGV[3] do local t=redis.call('time') redis.call('HSET','h:'..ARGV[1]..KEYS[1]..t[2]..'o'..index,'orderid',redis.call('SRANDMEMBER','charblocks'..KEYS[1])..t[2]..'o'..index,'status',redis.call('SRANDMEMBER','statuses'..KEYS[1]),'category',redis.call('SRANDMEMBER','productcats'..KEYS[1]),'color',redis.call('SRANDMEMBER','colors'..KEYS[1])..'_'..redis.call('SRANDMEMBER','alpha'..KEYS[1]),'price',(5.25*redis.call('SRANDMEMBER','day'..KEYS[1])),'quantity',(5*redis.call('SRANDMEMBER','day'..KEYS[1])),'builddate',redis.call('SRANDMEMBER','year'..KEYS[1])..'\-'..redis.call('SRANDMEMBER','month'..KEYS[1])..'\-'.. string.format('%02x',redis.call('SRANDMEMBER','day'..KEYS[1])),'timestamp',t[1]-(1000 * (redis.call('SRANDMEMBER','day'..KEYS[1]) * redis.call('SRANDMEMBER','day'..KEYS[1])))) end return (ARGV[3]-ARGV[2])..' '..KEYS[1]..' routed orders loaded with ending value of: '..ARGV[3]" 1 {1} ordersPA 1 250001
EVAL "for index=ARGV[2],ARGV[3] do local t=redis.call('time') redis.call('HSET','h:'..ARGV[1]..KEYS[1]..t[2]..'o'..index,'orderid',redis.call('SRANDMEMBER','charblocks'..KEYS[1])..t[2]..'o'..index,'status',redis.call('SRANDMEMBER','statuses'..KEYS[1]),'category',redis.call('SRANDMEMBER','productcats'..KEYS[1]),'color',redis.call('SRANDMEMBER','colors'..KEYS[1])..'_'..redis.call('SRANDMEMBER','alpha'..KEYS[1]),'price',(5.25*redis.call('SRANDMEMBER','day'..KEYS[1])),'quantity',(5*redis.call('SRANDMEMBER','day'..KEYS[1])),'builddate',redis.call('SRANDMEMBER','year'..KEYS[1])..'\-'..redis.call('SRANDMEMBER','month'..KEYS[1])..'\-'.. string.format('%02x',redis.call('SRANDMEMBER','day'..KEYS[1])),'timestamp',t[1]-(1000 * (redis.call('SRANDMEMBER','day'..KEYS[1]) * redis.call('SRANDMEMBER','day'..KEYS[1])))) end return (ARGV[3]-ARGV[2])..' '..KEYS[1]..' routed orders loaded with ending value of: '..ARGV[3]" 1 {2} ordersPA 1 250001
```

#### The following creates a search index: (adjust the prefix as fits your dataset)

```
FT.CREATE idx_paorders PREFIX 1 "h:order" ON HASH SCHEMA category TAG status TAG color TAG price NUMERIC SORTABLE quantity NUMERIC SORTABLE builddate TAG timestamp NUMERIC SORTABLE
```

#### FT.INFO can be useful to monitor progress of Search Index:
``` 
FT.INFO idx_paorders
```
#### the following are some sample queries:

``` 
FT.AGGREGATE idx_paorders "@timestamp:[-inf,1686104900] -@status:{cancelled}"  groupby 0 reduce count 0 as total_quantity_not_cancelled
```

```
FT.AGGREGATE idx_paorders "@timestamp:[-inf,1686004900] -@status:{cancelled}" groupby 0 reduce count 0 as total
```

```
FT.AGGREGATE idx_paorders "@timestamp:[-inf,1686004900] -@status:{cancelled}" groupby 1 @status reduce count 0 as total
```

```
FT.AGGREGATE idx_paorders "@timestamp:[-inf,1686004900] @status:{shipped}" groupby 0 reduce count 0 as total
```

``` 
FT.AGGREGATE idx_paorders "@timestamp:[-inf,1686004900] @color:{silver_K}"  groupby 2 @status @quantity reduce SUM 1 quantity as  total_quantity_by_LOT_AND_status
```

```
FT.AGGREGATE idx_paorders "@timestamp:[-inf,1686004900] -@status:{delivered}" groupby 1 @color reduce count 0 as total
```

#### This allows the observer to see inside the engine a bit:
```
FT.PROFILE idx_paorders AGGREGATE QUERY "@timestamp:[-inf,1686004900] -@status:{cancelled}" groupby 1 @color reduce count 0 as total
```
****
****
#### This next command is for Redis Enterprise and upgrades a database with the named search and json modules.  It also enables the use of configuration settings for TIMEOUT and such:
``` 
rladmin> upgrade module db_name db:44 module_name ReJSON version 20405 module_args "" module_name search version 20609 module_args "PARTITIONS AUTO MAXPREFIXEXPANSIONS 9000000000000000000 TIMEOUT 120000 ON_TIMEOUT FAIL"
```
#### These sample queries are for when ODBC or JDBC SQL drivers are used on top of the search index:
``` 
SELECT count(status) as total_quantity_for_status,status from idx_paorders where status NOT LIKE 'cancelled%' and timestamp < 1685470009 group by status;
```
``` 
SELECT count(status) as total_quantity_for_status,status from idx_paorders where status NOT LIKE 'cancelled%' and timestamp < 1685404900 group by status;
```
``` 
SELECT count(*) howmany from idx_paorders;
```