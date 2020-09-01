只有针对表级的只读权限

CREATE USER gp\_read WITH ENCRYPTED PASSWORD 'password';

grant select on dw.dw\_user\_login\_log  to gp\_read;  授予表权限

 grant all on SCHEMA dw to gp\_read;  授予架构权限

\du+

\dt

SELECT \* FROM pg\_roles;

