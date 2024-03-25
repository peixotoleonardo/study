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

## COPY

COPY copia arquivos ou diretórios do host e os adiciona para o filesystem
da imagem.

```dockerfile
COPY <src> ... <dest>
```
