# `s3-satis` tool

> ## Fork da MIDDAG
>
> Fork de [`kduma-OSS/CLI-s3-satis`](https://github.com/kduma-OSS/CLI-s3-satis). Upstream continua sendo a referência para uso e configuração; abaixo só o que muda aqui.
>
> **Por que o fork existe.** O Composer valida `config.github-oauth` contra um charset que **não inclui hífen**, e aborta o build inteiro se o token não casar. Installation tokens de GitHub App vêm com hífen, então o build morria na primeira linha — e como o satis roda com `--skip-errors`, o job saía **verde** sem regenerar um único arquivo de metadata, congelando o índice em silêncio. A validação foi **removida no Composer 2.10.0** (a 2.9.8 ainda a tinha). Este fork existe para carregar Composer >= 2.10.
>
> Cadeia que isso obriga: `composer/satis` `dev-main` → `composer/composer ^2.10.1` + `php ^8.4`, e o `symfony/console ^7.4.13` dele exige `laravel-zero/framework ^12.1` e `nunomaduro/termwind ^2.0`.
>
> **PHP 8.4 é piso.** Não é preferência — é o `require` do satis `dev-main`. Host com 8.2/8.3 não roda este PHAR.
>
> **Distribuição: R2 público, canal único.** O GitHub Release existe só como changelog, sem artefato. O PHAR sai no bucket `middag-public`, com sha256 e manifest:
>
> ```
> https://s3-middag-public.middag.com.br/s3-satis/<versão>/s3-satis.phar
> https://s3-middag-public.middag.com.br/s3-satis/<versão>/s3-satis.phar.sha256
> https://s3-middag-public.middag.com.br/s3-satis/manifest.json
> ```
>
> Versão publicada é **imutável**: o release aborta se o objeto já existir. Para resolver a mais recente, leia `latest` do manifest. Dois canais para o mesmo artefato divergem — foi por isso que o do Release foi cortado.
>
> **Trava de regressão.** `php.yml` e `release.yml` reprovam um PHAR cujo Composer embutido seja < 2.10.0 ou recuse um token com hífen. Sem ela, um bump de dependência reintroduz o bug em silêncio, que é o modo de falha mais caro deste projeto.

Tool to generate a [Composer](https://getcomposer.org/) PHP packages repository
(based on [Satis - static Composer repository generator](https://github.com/composer/satis))
and synchronize generated repository with a [Amazon S3](https://aws.amazon.com/s3/)
(or compatible) bucket.


Check full documentation here: [opensource.duma.sh/systems/serverless-satis/s3-satis](https://opensource.duma.sh/systems/serverless-satis/s3-satis)


## Setup

You can install `s3-satis` tool in four ways:

- As a [Docker](https://www.docker.com/) container -using image [ghcr.io/kduma-oss/s3-satis](https://github.com/kduma-OSS/CLI-s3-satis/pkgs/container/s3-satis)
- Global composer installation - tool will be available globally as `s3-satis` command
- You can download phar executable file from [GitHub Releases](https://github.com/kduma-OSS/CLI-s3-satis/releases/latest) page
- Download source code form [GitHub](https://github.com/kduma-OSS/CLI-s3-satis) to run

## Usage

First prepare a `satis.json` file with your repository configuration.
This tool is based on [Satis - static Composer repository generator](https://github.com/composer/satis){:target="_blank"},
so please check [Satis documentation](https://getcomposer.org/doc/articles/handling-private-packages-with-satis.md){:target="_blank"}
for configuration options.

```json
{
  "name": "my/repo",

  "homepage": "https://satis.example.com",

  "repositories": [
    { "type": "vcs", "url": "https://github.com/laravel/framework" }
  ],

  "require-all": true
}
```

Second, configure your environment variables (or `.env` file) with your S3 bucket credentials:
```env
S3_ACCESS_KEY_ID=
S3_SECRET_ACCESS_KEY=
S3_REGION=us-east-1
S3_BUCKET=
S3_ENDPOINT=
S3_USE_PATH_STYLE_ENDPOINT=false
```

Then run `s3-satis` tool to generate repository and upload it to S3 bucket:
```bash
s3-satis build satis.json
```
