# Rubocop ガイド

## 対象

 Ruby on Rails プロジェクト

## 目的と概要

コードレビューを行うにあたって、ブラケットの内側に半角スペースがないとか、ハッシュの記法が古いとか、そういった指摘があると、レビューする方もされる方も萎えるし時間がもったいない。この対策として、コーディング規約を設け、その準拠チェックを外部ツールに任せることにする。

そのツールが [Rubocop](https://github.com/bbatsov/rubocop) だ。Rubocop は、[The Ruby Style Guide](https://github.com/bbatsov/ruby-style-guide) と [The Rails Style Guide](https://github.com/bbatsov/rails-style-guide) のガイドラインに沿ってコーディングスタイルのチェックを行ってくれる。ここでは、Rubocop の導入と運用について、最低限の解説を行う。

## 導入

### Gemfile への追加

development グループに追加する。
```ruby
group :development do
  gem 'rubocop'
end
```

### .rubocop.yml の追加

標準では、1行あたりの文字数が80文字程度であったり、Ascii 以外のコメント(日本語等)が打てなかったり等、不便な点も多いので .rubocop.yml でカスタマイズする。プロジェクトのルートディクレクトリに、[.rubocop.yml](/.rubocop.yml) を追加する。

## 運用

### 手動実行

ターミナルから `rubocop` と叩くだけでチェックを行ってくれる。また、 `rubocop -a` ( `rubocop --auto-correct` ) と叩くと自動でコードの修正まで行ってくれる。ただし、意図しない修正が行われる可能性もあるので、auto-correct 前の指摘箇所の確認と、auto-correct 後の diff の確認は行うこと。

### 自動実行

一通りコードを書く度にターミナルを叩くのは面倒なので、Guard を使ってファイルを保存する度に Rubocop を自動実行して結果を通知する。Gemfile に [guard-rubocop](https://github.com/yujinakayama/guard-rubocop) と [terminal-notifier-guard](https://github.com/Codaisseur/terminal-notifier-guard) を追加する。
```ruby
group :development do
  gem 'rubocop'
  gem 'guard-rubocop'
  gem 'terminal-notifier-guard'
end
```

インストール後、Guardfile を作成する。
```console
guard init rubocop
```

Guardfile を開くと、下記のようなコードが生成されている。
```ruby
guard :rubocop do
  watch(%r{.+\.rb$})
  watch(%r{(?:.+/)?\.rubocop\.yml$}) { |m| File.dirname(m[0]) }
end
```

自動チェックの結果を通知センターに表示するために、1行目に `notification: true` オプションを付加する。
```ruby
guard :rubocop, notification do
  watch(%r{.+\.rb$})
  watch(%r{(?:.+/)?\.rubocop\.yml$}) { |m| File.dirname(m[0]) }
end
```

Guard を実行する。制御は返ってこないので、別タブを開いて実行すると良い。
```console
bundle exec guard
```

早速 Guardfile の正規表現のコーディングスタイルについて指摘を受けるので、 `rubocop --auto-correct` で修正すると良い。最終的な Guardfile はこのようになる。
```ruby
guard :rubocop, notification: true do
  watch(/.+\.rb$/)
  watch(%r{(?:.+/)?\.rubocop\.yml$}) { |m| File.dirname(m[0]) }
end
```
