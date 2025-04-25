# Odoo Docker
This setup is using official image [Odoo 18](https://hub.docker.com/_/odoo)  and [PostgreSQL 17](https://hub.docker.com/_/postgres). I follow the documentation [here](https://opensourcehustle.com/blog/odoo-erp-1/odoo-windows-docker-compose-7) to install it on my system. Make sure you install [docker](https://docs.docker.com/engine/install/) on your system to follow this guide.

# Folder Structure
```
odoo-docker/ 
├── addons/ *# Custom addons here*
│ └── <your_addon> *# example of custom addons*
│ ├── **init**.py 
│ └── **manifest**.py 
├── config/ *# Odoo configuration*
│ └── odoo.conf 
├── docker-compose.yml  *# Main docker file*
├── README.md
```

# Docker Compose Structure

    services:
    db:
    image: postgres:17
    ports:
    - "5432:5432"
    environment:
    - POSTGRES_USER=odoo
    - POSTGRES_PASSWORD=odoo
    - PGDATA=/var/lib/postgresql/data/pgdata
    volumes:
    - odoo-db-data:/var/lib/postgresql/data
    
    odoo18:
    image: odoo:18.0
    depends_on:
    - db
    ports:
    - "8068:8069"
    volumes:
    - odoo-web-data:/var/lib/odoo
    - ./config:/etc/odoo
    - ./addons:/mnt/extra-addons
    command: odoo -d odoo_db -i stock --without-demo=all --db_user=odoo --db_password=odoo --db_host=db
    
    volumes:
    odoo-db-data:
    odoo-web-data:

I only have only have one instance of Odoo application since i don't need to use more than one version of the Odoo app. If you want to follow the original tutorial, you can check[here](https://opensourcehustle.com/blog/odoo-erp-1/odoo-windows-docker-compose-7).

# Odoo.conf directives
Copy and paste this directives into **./config/odoo.conf**

    [options]  
    addons_path = /mnt/extra-addons  
    data_dir = /var/lib/odoo

# The commands from the [original tutorials](https://opensourcehustle.com/blog/odoo-erp-1/odoo-windows-docker-compose-7)

## Launch Odoo/Postgres using Docker Compose
    docker compose up -d
    
## Restarting Odoo Container and Useful Flags
If you are using this setup to work on a custom Odoo module then you will need to restart the container for changes to take affect, but before restarting there are a few useful flags that you could could add to the docker compose file, first is the _-u_ flag_,_ this flag allows you to update a specific module on restart, for example, using the custom website module provided above, the command will update to:

    command: odoo -d odoo_db -u hustle_website --db_user=odoo --db_password=odoo --db_host=db

This would update the hustle_website module on restart.

> **NOTE:** 
> I removed the  _-i stock --without-demo=all_ flag from the command as this flag recreates the database,  **erasing all progress**, this flag should only be used once to create the database then removed.

Another useful flag that you can add is the  _--dev=xml_ flag, when using this flag you do not need to restart the container when making changes to XML files, simply refresh the page and the changes are in affect, here is the command with the XML flag:

    command: odoo -d odoo_db -u hustle_website --dev=xml --db_user=odoo --db_password=odoo --db_host=db  

Taking the above in consideration we can restart the Odoo container by running the following command:

    docker compose restart odoo18

We do not need to restart the database, Odoo container alone will suffice, therefor specify the Odoo container name after the restart option to restart it.

## Cleaning Up the Docker Environment
In case you want to clean up the environment, you can run the following command which will remove all the containers:

    docker compose down

If you want to remove the associated volumes with the containers you can add the  _-v_ flag:

    docker compose down -v

Incase you removed the containers but not the volumes, then you can remove the volumes by running:

    docker volume rm <volume name>

If you do not know the volume name then you can list them by running:

    docker volume list

Or simply use the Docker Desktop application to do all of the above.
