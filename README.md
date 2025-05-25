# Keycloak

## Instalación

La instalación depende de si el keycloak que vamos a instalar por primera vez o es una actualización de keycloak.

### Instalación nueva

```shell
kubectl apply -f alm
```

## BackUp del fichero de configuración
```shell
kubectl exec -n keycloak -it keycloak-0 -- cat /opt/keycloak/conf/keycloak.conf > keycloak.conf
```

### Actualizar Keycloak

En la mayoría de casos, solo es poner el nuevo tag de imagen en el `alm/statefulset-keycloak.yaml` y hacer el apply. Pero sobretodo lo que hay que mirar son todas las **release notes** de la nueva version y las intermedias que pueda haber, por si hay algún breaking change, o hay que poner antes una actualización intermedia, o seguir algún procedimiento especial..

### Migración de instancia de Keycloak (y/o actualizar version major de PostgreSQL)

Lo único que hay que migrar es la BDD de PostgreSQL. Tras ello se debe exportar la base de datos antigua para añadirla a la nueva instancia. Para realizar la exportación se debe conectar a la base de datos antigua y ejecutar el siguiente comando:

```shell
# Un kubectl exec (o usar openlens) para acceder dentro del contenedor de postgres (antiguo)
pg_dump -U <usuario_base_datos> -h <host_base_datos> -d <nombre_base_datos> -W -f /tmp/backup.sql
# podemos copiar el backup en nuestro pc (por ejemplo), con un kubectl cp, por ejemplo

## Otra opción es conectar el disco nuevo al contenedor en un directorio X y dejar ya copiado el backup
```


Se importa la base de datos antigua en la nueva base de datos:
```shell
# Arrancado el nuevo PostgreSQL que estará vació, podemos hacer el kubectl cp de nuestro pc al contenedor y luego hacer el import.
psql -U <usuario_base_datos> -h <host_base_datos> keycloak < /tmp/backup.sql
```

Ahora ya se puede arrancar keycloak:
```shell
kubectl apply -f ./alm/statefulset-keycloak.yaml
kubectl apply -f ./alm/service-keycloak.yaml
kubectl apply -f ./alm/ingress.yaml
```