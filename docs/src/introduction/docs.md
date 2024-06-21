# Documentation [文档]

If you want to locally build this documentation, you'll have to download [mdBook](https://github.com/rust-lang/mdBook), [mdBook-admonish](https://github.com/tommilligan/mdbook-admonish) and [mdBook-mermaid](https://github.com/badboy/mdbook-mermaid) and add their parent directories to the `PATH`env variable so that the executables are found.

[ 如果您想在本地构建此文档，则必须下载 [mdBook](https://github.com/rust-lang/mdBook), [mdBook-admonish](https://github.com/tommilligan/mdbook-admonish) 和 [mdBook-mermaid](https://github.com/badboy/mdbook-mermaid) 并将它们的父级目录添加到系统环境变量 PATH 中，以便找到可执行文件]

You can do this via bash (after running `source setup.sh`):

[ 您可以通过 bash 执行此操作（创建setup.sh文件, 然后复制粘贴下面的代码并保存, 然后运行 source setup.sh）]

```bash
export MDBOOK_VERSION="0.4.28"
export MDBOOK_ADMONISH_VERSION="1.9.0"
export MDBOOK_MERMAID_VERSION="0.12.6"
export MDBOOK_SITEMAP_VERSION="0.1.0"
curl -L https://github.com/rust-lang/mdBook/releases/download/v$MDBOOK_VERSION/mdbook-v$MDBOOK_VERSION-x86_64-unknown-linux-gnu.tar.gz | tar xz -C ${REPO_ROOT}/tools
curl -L https://github.com/tommilligan/mdbook-admonish/releases/download/v$MDBOOK_ADMONISH_VERSION/mdbook-admonish-v$MDBOOK_ADMONISH_VERSION-x86_64-unknown-linux-gnu.tar.gz | tar xz -C ${REPO_ROOT}/tools
curl -L https://github.com/badboy/mdbook-mermaid/releases/download/v$MDBOOK_MERMAID_VERSION/mdbook-mermaid-v$MDBOOK_MERMAID_VERSION-x86_64-unknown-linux-gnu.tar.gz | tar xz -C ~/tools
export PATH=${REPO_ROOT}/tools:$PATH
```

You then can just run the following to build the documentation in html format:

[ 然后，您可以运行以下命令来构建 html 格式的文档：]

```bash
./docs.sh
```

The documentation will then be built in docs/book.

[ 文档将构建在docs/book中]