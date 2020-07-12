---
layout: post
title: "Erros na atualização de versões, ou recuperação de backup de banco de dados do Zabbix"
date: 2020-07-12 17:11:19 -0300
categories:
---

Em resumo, é preciso executar o processo do zabbix_server após atualização ou restauração do banco de dados ANTES de acessar o Frontend PHP no browser.

### O problema recorrente

Participo do grupo de [Telegram do Zabbix Brasil][zbx-telegram] e várias vezes vejo uma pergunta sobre o mesmo erro quando se tenta fazer a atualização (ou restauração de backup) de versão mais antiga para uma versão mais nova da ferramenta de monitoração [Zabbix][zbx-site], erro esse que ocorre quando se tenta acessar o Frontend PHP:

> "Database error: The frontend does not match Zabbix database. Current database version (mandatory/optional): 4020000/4020001. Required mandatory version: 4040000. Contact your system administrator."

### "Entender o problema é igual a saber 50% da solução"

Esse erro normalmente acontece porque o Zabbix tem no seu banco de dados uma tabela com um identificador de versão do banco, esse parâmetro pode ser alterado pelo daemon do zabbix_server quando o mesmo inicia. O Frontend PHP tem o mesmo identificador fixo no código de acordo com a versão, e ao ser acesso via web browser ele compara a sua versão Major (Os três dígitos iniciais) com a do Zabbix Server. Alguns exemplos de identificador de versão:

- Versão 3.4: 3040000
- Versão 4.0: 4000000
- Versão 4.4: 4040000
- Versão 5.0: 5000000

Então no caso em que o identificador do banco de dados esteja na versão 4.4 (Id 4040000) e ocorra atualização da ferramenta do Zabbix para a versão 5.0 (Id 5000000). Ao acessar (sem executar o daemon do zabbix_server) o Frontend PHP, ele vai comparar a versão Major e ver que são diferentes e vai apresentar o famigerado erro.

**Para resolver o problema** é preciso executar o processo do zabbix_server pois ele vai identificar a versão mais antiga no banco, fazer as atualizações necessárias na estrutura desse último e depois disso vai atualizar o identificador de versão.

### Passo a passo de como atualizar corretamente a versão do Zabbix

1. Pare o processo do httpd (Apache ou Nginx);
2. Pare o processo do zabbix_server;
3. Faça _backup_ do banco de dados do Zabbix;
4. Realize a atualização dos pacotes da ferramenta Zabbix usando o método necessário (apt, yum, 1. compilação...);
5. Inicie o zabbix_server e espere ele atualizar o banco de dados (verifique nas logs ou no status do 1. serviço, quanto maior o banco mais tempo leva);
6. Inicie o processo do httpd (Apache ou Nginx);
7. Agora o frontend deve funcionar sem erros.

Se o inglês não for barreira, você pode ver [este vídeo oficial][zbx-install] da ferramenta sobre o processo e dicas de upgrade.

### Não existe downgrade da versão do banco de dados do Zabbix

FAÇA SEU BACKUP! [Conforme post do Dimitri][zbx-forum] (Desenvolvimento e suporte oficial da Zabbix Inc.) nos fóruns oficiais, não existe procedimento para mudar o banco de dados para uma versão anterior, eles oferecem o suporte para esse tipo de situação mediante contratação oficial paga.

[zbx-telegram]: https://t.me/ZabbixBrasil
[zbx-site]: https://www.zabbix.com/
[zbx-install]: https://www.youtube.com/watch?v=Meyiiht5WxE
[zbx-forum]: https://www.zabbix.com/forum/zabbix-troubleshooting-and-problems/375864-downgrade-zabbix-database-version?p=376141#post376141
