
### Buscar todas las llamadas a \*Error\* dentro de varios proyectos:

    $ find . -maxdepth 2 -name "src" -print0 | xargs -0 -n1 -I{}  grep -rhEzo "[a-zA-Z]*Error[a-zA-Z]*\([^\)]*[\"\'\`]+([^\)])*\)" {} | tr "\0" "\n"
