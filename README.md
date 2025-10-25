# Repositorio de Configuración Centralizada

Este repositorio contiene la configuración centralizada para todos los microservicios del sistema de transporte, gestionada por Spring Cloud Config Server.

## Estructura

```
transport-config-repo/
├── rs-auth/
│   └── application.yml          # Configuración de rs-auth
├── rs-dispatcher/
│   └── application.yml          # Configuración de rs-dispatcher
├── rs-api-gateway/
│   └── application.yml          # Configuración de rs-api-gateway
└── application.yml              # Configuración compartida por todos
```

## Uso

### Configuración del Config Server

El Config Server (`rs-config-server`) está configurado para leer desde este repositorio:

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/<OWNER>/transport-config-repo.git
          default-label: main
          search-paths: '{application}'
```

### Perfiles

- **Desarrollo local**: El Config Server puede usar `spring.profiles.active=native` y leer archivos del classpath
- **Producción**: Usa `spring.profiles.active=git` para leer de este repositorio GitHub

### Secrets

⚠️ **IMPORTANTE**: NO commits secrets en texto plano.

Para manejar secrets:
1. Usa variables de entorno: `${DB_PASSWORD}` o `${JWT_SECRET}`
2. Usa Spring Cloud Vault o AWS Secrets Manager
3. Cifra valores sensibles con `spring-cloud-config encrypt/decrypt`

### Ejemplo de cifrado

```bash
# Cifrar un valor
curl http://localhost:8888/encrypt -d mysecret

# En application.yml
jwt:
  secret: '{cipher}AQATLxvdnL...'
```

## Endpoints del Config Server

- `http://localhost:8888/{application}/{profile}` - Obtener configuración
- `http://localhost:8888/{application}-{profile}.yml` - YAML
- `http://localhost:8888/rs-auth/default` - rs-auth en perfil default

## Refresh

Para refrescar configuración sin reiniciar:

```bash
curl -X POST http://localhost:8081/actuator/refresh
```

## Orden de arranque recomendado

1. rs-eureka-server
2. rs-config-server
3. rs-auth, rs-dispatcher, rs-api-gateway
