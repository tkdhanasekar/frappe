### Frappe ERPNext Docker Installation
update the server
```
apt update
```
install the docker and dependencies
```
apt install docker.io containerd docker-buildx docker-compose-v2 docker-compose openssl -y
```
clone the frappe_docker repo
```
git clone https://github.com/frappe/frappe_docker
```
navigate to the frappe_docker directory
```
cd frappe_docker
```
create a gitops directory
```
mkdir ~/gitops
```
configure the traefik.env file
```
echo 'TRAEFIK_DOMAIN=traefik.hashlabs.in' > ~/gitops/traefik.env
```
```
echo 'EMAIL=tkdana@gmail.com' >> ~/gitops/traefik.env
```
```
echo 'HASHED_PASSWORD='$(openssl passwd -apr1 Passw0rd | sed 's/\$/\\\$/g') >> ~/gitops/traefik.env
```
Deploy the traefik container with letsencrypt SSL
```
docker compose --project-name traefik --env-file ~/gitops/traefik.env -f overrides/compose.traefik.yaml up -d
```
```
docker compose --project-name traefik --env-file ~/gitops/traefik.env -f overrides/compose.traefik.yaml -f overrides/compose.traefik-ssl.yaml up -d
```
Install and configure MariaDB
```
echo "DB_PASSWORD=StrongPassw0rd" > ~/gitops/mariadb.env
```
```
docker compose --project-name mariadb --env-file ~/gitops/mariadb.env -f overrides/compose.mariadb-shared.yaml up -d
```
Install ERPNext
```
cp example.env ~/gitops/erpnext-one.env
```
```
sed -i 's/DB_PASSWORD=123/DB_PASSWORD=StrongPassw0rd/g' ~/gitops/erpnext-one.env
```
```
sed -i 's/DB_HOST=/DB_HOST=mariadb-database/g' ~/gitops/erpnext-one.env
```
```
sed -i 's/DB_PORT=/DB_PORT=3306/g' ~/gitops/erpnext-one.env
```
```
echo 'ROUTER=erpnext-one' >> ~/gitops/erpnext-one.env
```
```
echo "SITES=\`erp1.hashlabs.in\`,\`erp2.hashlabs.in\`" >> ~/gitops/erpnext-one.env
```
```
echo "BENCH_NETWORK=erpnext-one" >> ~/gitops/erpnext-one.env
```
```
cd && cd gitops
```
add Lets encrypt mail id in the erpnext-one.env file
```
vim erpnext-one.env
```
```
LETSENCRYPT_EMAIL=tkdana@gmail.com
```
```
cd && cd frappe_docker
```
```
docker compose --project-name erpnext-one --env-file ~/gitops/erpnext-one.env -f compose.yaml -f overrides/compose.redis.yaml -f overrides/compose.multi-bench.yaml config > ~/gitops/erpnext-one.yaml
```
```
docker compose --project-name erpnext-one --env-file ~/gitops/erpnext-one.env -f compose.yaml -f overrides/compose.redis.yaml -f overrides/compose.multi-bench.yaml -f overrides/compose.multi-bench-ssl.yaml config > ~/gitops/erpnext-one.yaml
```
```
docker compose --project-name erpnext-one -f ~/gitops/erpnext-one.yaml up -d
```
```
docker compose --project-name erpnext-one exec backend   bench new-site erp1.hashlabs.in --mariadb-root-password StrongPassw0rd --install-app erpnext --admin-password StrongPassw0rd
```
```
docker compose --project-name erpnext-one exec backend   bench new-site erp2.hashlabs.in --mariadb-root-password StrongPassw0rd --install-app erpnext --admin-password StrongPassw0rd
```

