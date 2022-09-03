# openssl-cheatsheet

OpenSSL Commands Cheat Sheet

[![CC BY-NC-SA 4.0][cc-by-nc-sa-shield]][cc-by-nc-sa]

## Generate RSA Private Key

Generate RSA Private Key with 4096 bits and save as `key.pem`

```
openssl genrsa \
    -out key.pem \
    4096
```

## Tips

OpenSSL is built as command and subcommand

```
openssl subcommand
```

Every subcommand has its own man page

```
man openssl-subcommand
```

Every argument is defined using a single dash

```
openssl subcommand -argument
```

## License

This work is licensed under a
[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License][cc-by-nc-sa].

[![CC BY-NC-SA 4.0][cc-by-nc-sa-image]][cc-by-nc-sa]

[cc-by-nc-sa]: http://creativecommons.org/licenses/by-nc-sa/4.0/
[cc-by-nc-sa-image]: https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png
[cc-by-nc-sa-shield]: https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg
