# MongoDB

３台構成のレプリケーション構成や４台での優先度付けした場合等の実運用について、Gitlabに検証内容や結果を記載しているので、
公開できる情報に修正してここに書いていきます。

→ 社内Gitlabにまとめているので公開できる形にして公開しようと思います。


## Primary/Secondaryの重み付け設定をする

- Priorityの設定
  - 0 - 1000 の値で値が高い方が優先してプライマリに昇格する
  - 優先度が高いほうがプライマリとして動作する

- Configをセット
`> cfg = rs.conf()`

- 配列の数字は_idを入れる( 0は絶対にPrimaryにならない状態になる)
- プライオリティを10にセット
`> cfg.members[1].priority = 50`

- 設定を反映する
`> rs.reconfig(cfg)`

- 反映後の設定を確認する
`> rs.conf()`

```
testRep1:SECONDARY> rs.conf()
{
	"_id" : "testRep1",
	"version" : 11,
	"members" : [
		{
			"_id" : 0,
			"host" : "172.31.2.198:27017",
			"arbiterOnly" : true
		},
		{
			"_id" : 1,
			"host" : "172.31.7.138:27017",
			"priority" : 50
		},
		{
			"_id" : 2,
			"host" : "172.31.2.117:27017",
			"priority" : 30
		}
	]
}
```

## MongoDBのリングにノードを追加・削除する

- 構成からノードを外すコマンド(MongoDBのプライマリで実行
  - `>rs.remove("<MongoDBのIPアドレス>:27017");`
- 追加したノードをセカンダリから同期をする場合
  - `> SECONDARY> rs.syncFrom("<MongoDBのIPアドレス>:27017")`

- 不要になるMongoDBを切り離す
  - 下記のコマンドはPrimaryで実行する
  - `$ rs.remove("<MongoDBのIPアドレス>:27017");`

## oplogについて

- oplogを確認する
`PRIMARY> db.getReplicationInfo()`
