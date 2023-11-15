### 编写compose
``` yaml
version: "3"
services:
  postgres:
    container_name: postgres
    image: postgres:12-alpine
    restart: always
    environment:
      POSTGRES_USER: <username>
      POSTGRES_PASSWORD: <password>
      POSTGRES_DB: <database_name>
    ports:
      - 5432:5432
    volumes:
      - postgres-data:/var/lib/postgresql/data
volumes:
  postgres-data: { }
```

### 更改时区
``` sql
ALTER DATABASE "<database_name>" SET timezone TO 'Asia/Shanghai';
```