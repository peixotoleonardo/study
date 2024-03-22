# Referência Dockerfile

## WORKDIR

```dockerfile
WORKDIR /path/to/workdir
```

Configura o diretório de trabalho para qualquer instrução abaixo:

- RUN
- CMD
- ENTRYPOINT
- COPY
- ADD

Se o diretório não existir, ele será criado.