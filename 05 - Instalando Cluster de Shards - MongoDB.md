# Instalando Cluster de Shards - MongoDB
Nos últimos dias recebi um desafio bem legal de instalar um cluster de shards do MongoDB, e, como sou bem iniciante nesse sentido, aceitei o desafio. Aqui, vai mais como anotação mesmo de como fazer, e vou seguir as seguintes premissas:

- Serão três instâncias que farão todo o trabalho;
- Uma delas será o balancer e router, as outras duas que irão armazenar os dados;
- Caso umas das instâncias de armazenamento caia, a outra deve assumir sem nenhum problema;


## Preparativos
Precisaremos de três máquinas físicas, vamos nomeá-las da seguinte forma:

- Máquina 1 - `mongo0.maxnovais.me`
- Máquina 2 - `mongo1.maxnovais.me`
- Máquina 3 - `mongo2.maxnovais.me`

Todas as máquinas já estarão com CentOS 8, MongoDB e mais algumas ferramentas usuais desses servidores. Existe um macete, que acabei tendo problemas, que é a questão de `IPV4` físico, existe alguns problemas em trabalhar com eles no Mongo. Se **NÃO** estiver usando um VPS que permita o uso de subdomains e domains, sugiro em editar o `/etc/hosts` das três máquinas:

```
# vim /etc/hosts
10.1.1.1    mongo0.maxnovais.me
10.1.1.2    mongo1.maxnovais.me
10.1.1.3    mongo2.maxnovais.me
```

## Estrutura
A estrutura que iremos usar segue dessa forma:

```
+ mongo0.maxnovais.me (essa instância não armazena dados)
|-+ Query Routers
| |- porta 27017 (padrão do mongo)
| |- porta 26061
|-+ Config Servers
| |- porta 26050
| |- porta 26051
| |- porta 26052
|-+ Arbiter Shard
| |- porta 27000
+ mongo1.maxnovais.me
|-+ Shards (ReplicaSet 'a')
| |- porta 27000 - prioridade 1.0
| |- porta 27001 - prioridade 0.9
| |- porta 27002 - prioridade 0.8
+ mongo2.maxnovais.me
|-+ Shards (ReplicaSet 'a')
| |- porta 27000 - prioridade 0.7
| |- porta 27001 - prioridade 0.6
| |- porta 27002 - prioridade 0.5
```

**Observação:** Eu vou tentar ser o mais claro possível, em dizer qual das máquinas está rodando e será executado cada passo dessa anotação.


## Lidando com os config server
Primeiro, vamos acessar a máquina `mongo0.maxnovais.me` e criar a seguinte estrutura:
```
/var/lib/mongodb/shardedreplic
|- cfg0
|- cfg1
|- cfg2
```
Após criada essa estrutura, vamos rodar três comandos para iniciar os servidores de configuração do mongo:

- `# mongod --configsvr --port 26050 --logpath /var/lib/mongodb/shardedreplic/log.cfg0 --logappend --dbpath /var/lib/mongodb/shardedreplic/cfg0 --fork`
- `# mongod --configsvr --port 26051 --logpath /var/lib/mongodb/shardedreplic/log.cfg1 --logappend --dbpath /var/lib/mongodb/shardedreplic/cfg1 --fork`
- `# mongod --configsvr --port 26052 --logpath /var/lib/mongodb/shardedreplic/log.cfg2 --logappend --dbpath /var/lib/mongodb/shardedreplic/cfg2 --fork`


## Criando os fragmentos
Essa segunda parte, tem que ser feita nas duas máquinas `mongo1.maxnovais.me` e `mongo2.maxnovais.me`, vamos criar a mesma estrutura abaixo em ambas as máquinas:
```
/var/lib/mongodb/shardedreplic
|- a0
|- a1
|- a2
```

Após criada a estrutura, vamos subir os servidores, esse comando também deve ser executado nas duas máquinas:

- `# mongod --shardsvr --replSet a --dbpath /var/lib/mongodb/shardedreplic/a0 --logpath /var/lib/mongodb/shardedreplic/log.a0 --port 27000 --fork --logappend --smallfiles --oplogSize 50`
- `# mongod --shardsvr --replSet a --dbpath /var/lib/mongodb/shardedreplic/a1 --logpath /var/lib/mongodb/shardedreplic/log.a1 --port 27001 --fork --logappend --smallfiles --oplogSize 50`
- `# mongod --shardsvr --replSet a --dbpath /var/lib/mongodb/shardedreplic/a2 --logpath /var/lib/mongodb/shardedreplic/log.a2 --port 27002 --fork --logappend --smallfiles --oplogSize 50`


## Criando o juiz
Para criação da máquina arbiter, vamos usar a máquina `mongo0.maxnovais.me`, vamos criar a pasta na seguinte estutura:
```
/var/lib/mongodb/shardedreplic
|- arbiter
```

Vamos iniciar o shard de arbiter com o comando abaixo:

- `# mongod --shardsvr --replSet a --dbpath /var/lib/mongodb/shardedreplic/arbiter --logpath /var/lib/mongodb/shardedreplic/log.a0 --port 27000 --fork --logappend --smallfiles --oplogSize 50`

## Juntando as peças
Só revisando o que temos aqui até o momento:
```
mongo0.maxnovais.me
|- config server [0, 1 e 2]
|- arbiter

mongo1.maxnovais.me
|- shards [0, 1 e 2]

mongo2.maxnovais.me
|- shards [0, 1 e 2]
```

Agora vamos, da máquina `mongo1.maxnovais.me`, executar os comandos para entrar no mongo:
```
$ mongo --port 27000
```

Dentro do mongo, vamos iniciar a configuração.
```
> rs.initiate()
{
	"info2" : "no configuration explicitly specified -- making one",
	"me" : "mongo1.maxnovais.me:27000",
	"info" : "Config now saved locally.  Should come online in about a minute.",
	"ok" : 1
}
a.PRIMARY
```

Nesse momento iremos adicionar todos os shards disponíveis.
```
a:PRIMARY> rs.add("mongo1.maxnovais.me:27001")
{ "ok" : 1 }
a:PRIMARY> rs.add("mongo1.maxnovais.me:27002")
{ "ok" : 1 }
a:PRIMARY> rs.add("mongo2.maxnovais.me:27000")
{ "ok" : 1 }
a:PRIMARY> rs.add("mongo2.maxnovais.me:27001")
{ "ok" : 1 }
a:PRIMARY> rs.add("mongo2.maxnovais.me:27002")
{ "ok" : 1 }
a:PRIMARY>
```

Vamos adicionar o arbiter também a essa configuração:
```
a:PRIMARY> rs.addArb("mongo0.maxnovais.me:27000")
{ "ok" : 1 }
```

## Definindo prioridades / Editando Hosts
No mongo, o comando `rs.conf()` mostra a configuração atual da réplica criada, seus shards e suas posições.
Em qualquer momento, podemos alterar as configurações, definindo novos hosts, ou prioridade. Comunente, quando digitamos pela primeira vez isso, irá vir algo assim:
```
a.PRIMARY> rs.conf()
{
	"_id" : "a",
	"version" : 1,
	"members" : [
		{
			"_id" : 0,
			"host" : "172.168.1.1:27000"
		},
		{
			"_id" : 1,
			"host" : "mongo1.maxnovais.me:27001",
		},
		{
			"_id" : 2,
			"host" : "mongo1.maxnovais.me:27002",
		},
		{
			"_id" : 3,
			"host" : "mongo2.maxnovais.me:27000",
		},
		{
			"_id" : 4,
			"host" : "mongo2.maxnovais.me:27001",
		},
		{
			"_id" : 5,
			"host" : "mongo2.maxnovais.me:27002",
		},
		{
			"_id" : 6,
			"host" : "mongo0.maxnovais.me:27000",
			"arbiterOnly" : true
		}
	]
}
```

Vamos definir a prioridade das máquinas e mudar o host da máquina com id 0, primeiro instanciaremos o `rs.conf()` e após isso ele será tratado como um objeto dentro do Mongo.
Só atribuir o valor ao objeto com o comando abaixo:
```
a.PRIMARY> cfg = rs.conf()
a.PRIMARY> cfg.members[0].host = 'mongo1.maxnovais.me:27000'
```

Agora vamos colocar prioridade em todas as máquinas. Por padrão, as máquinas estão com valor 1 de prioridade, em decimal, a chave `priority` serve para isso.
Vale lembrar, que os shards com prioridade 1 não precisam ser delegados aqui, e também não será mostrado o atributo de prioridade no `rs.conf()`
```
a.PRIMARY> cfg.members[1].priority = 0.9
a.PRIMARY> cfg.members[2].priority = 0.8
a.PRIMARY> cfg.members[3].priority = 0.7
a.PRIMARY> cfg.members[4].priority = 0.6
a.PRIMARY> cfg.members[5].priority = 0.5
```

Ou seja, em ordem de escala, a instância `mongo2.maxnovais.me:27002` será a última a ser eleita pelo arbiter como primary.
Após isso, vamos salvar as edições e reconfigurar.
```
a.PRIMARY> rs.reconfig(cfg)
```


## Subindo os query routers
Por fim, devemos criar agora o query router, aquele que irá conectar a todos os shards e não armazenará nada.
Podemos seguir o processo na máquina `mongo0.maxnovais.me`, rodando as linhas abaixo pra iniciar o servidor:

`# mongos --configdb mongo0.maxnovais.me:26050,mongo0.maxnovais.me:26051,mongo0.maxnovais.me:26052 --fork --logappend --logpath /var/lib/mongodb/shardedreplic/log.mongos0`
`# mongos --configdb mongo0.maxnovais.me:26050,mongo0.maxnovais.me:26051,mongo0.maxnovais.me:26052 --fork --logappend --logpath /var/lib/mongodb/shardedreplic/log.mongos1 --port 26061`

Agora, vamos rodar entra no mongo com o comando abaixo.
*Nota:* Não precisa utilizar a porta, pois iremos entrar no processo do Query Router que esta na porta padrão (27017).
```
$ mongo
```

Podemos usar o comando `sh.status()` para ver como estão as coisas nesse processo:
```
mongos> sh.status()
--- Sharding Status --- 
sharding version: {
"_id" : 1,
"version" : 4,
"minCompatibleVersion" : 4,
"currentVersion" : 5,
"clusterId" : ObjectId("548eb941261fb6e96e17d275")
}
shards:
databases:
{ "_id" : "admin", "partitioned" : false, "primary" : "config" }
 
mongos>
``` 

Vamos adicionar o shard principal com o comando `sh.addShard()` e ver se está em load balancer `sh.getBalancerState()`:
```
mongos> sh.addShard("a/mongo1.maxnovais.me:27000")
{ "shardAdded" : "a", "ok" : 1 }
mongos> sh.getBalancerState()
true
```

## Referência:

- [Mongo DB Docs](https://docs.mongodb.com/manual/tutorial/deploy-shard-cluster/)
- [Mongo DB Spain](http://www.mongodbspain.com/en/2015/01/26/how-to-set-up-a-mongodb-sharded-cluster/)
