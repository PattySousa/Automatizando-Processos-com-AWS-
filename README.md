# Automatizando-Processos-com-AWS-
Criação de Tópico e SNS e criação de fila SQS padrão e DLQ na Nuvem AWS


Parte 1 => Criação dos Filas SQS

Acessar Console SQS: Faça login no Console AWS e navegue até o serviço SQS (Simple Queue Service). Criar Fila Dead-Letter (DLQ):
Clique em Criar fila. • Tipo: Selecione Padrão. Nome: Insira um nome único, ex: minha-dlq-lab-seunome-aaaammdd.
Configurações: Mantenha as configurações padrão por enquanto (ex: Período de retenção da mensagem, Tempo limite de visibilidade). Não configure um DLQ para o próprio DLQ nesta etapa. Clique em Criar fila. Após a criação, selecione a fila recém-criada na lista. Na aba Detalhes, localize e Copie o ARN da DLQ. Guarde este ARN, você precisará dele em breve.

![image](https://github.com/user-attachments/assets/c4d076ff-1799-4cf9-b689-15558be35b35)

Criar Fila Principal:
Volte para a página principal do SQS (clicando em “Filas” no menu
esquerdo) e clique em Criar fila. Tipo: Selecione Padrão. Nome: Insira um nome único, ex: minha-fila-principal-lab-eunome-aaaammdd. Configurações: Role para baixo até a seção Fila de mensagens mortas (Dead-letter queue). Marque a opção Habilitado. Escolha a fila de mensagens mortas: Cole o ARN da DLQ que você copiou no passo anterior. Número de coletas: Defina um valor máximo baixo para facilitar o teste, por exemplo, 3. Isso significa que se uma mensagem para recebida (polled) 3 vezes sem ser restaurada com sucesso, será ela movida para a DLQ. Mantenha as demais configurações padrão ou ajuste conforme necessário. Clique em Criar fila. Após a criação, selecione a fila principal na lista. Na aba Detalhes, localize e Copie o ARN da fila principal. Guarde este ARN, você precisará futuramente.

![image](https://github.com/user-attachments/assets/8ac0bcd9-e7dd-4e2b-b7de-e20a8a59e438)



Parte 2 => Criação do Tópico SNS

Acessar Console SNS: Navegue até o serviço SNS (Simple Notification Service). Criar Tópico: No menu esquerdo, clique em Tópicos. Clique em Criar tópico. Tipo: Selecione Padrão.
Nome: Insira um nome único, ex: meu-topico-lab-seunome-aaaammdd. Política de acesso: Deixe como Básico por enquanto. A permissão para SQS será adicionada na inscrição. Clique em Criar tópico. Após a criação, na página de detalhes do tópico, Copie o ARN do tópico. Guarde este ARN.

![image](https://github.com/user-attachments/assets/584ca091-606f-46bf-81a2-39fb36f9ac23)

Parte 3 => Conectando SNS e SQS

Inscrever Fila SQS no Tópico SNS: Ainda no console SNS, com seu item selecionado, vá para a aba Assinaturas. Clique em Criar assinatura. ARN do Tópico: Deve ser preenchido com o ARN do seu tópico. Protocolo: Selecione Amazon SQS. Endpoint: Cole o ARN da sua Fila Principal (criada no passo 3). (Opcional, recomendado para simplificar) Habilitar entrega de mensagens brutas (Enable raw message delivery): Marque esta caixa se desejar que a mensagem chegue na fila SQS exatamente como foi publicada no SNS, sem o metadado extra do SNS. Para este laboratório, você pode deixar desmarcado para ver a estrutura completa. Clique em Criar assinatura. Nota: A criação da assinatura geralmente configura automaticamente a política de acesso do fila SQS para permitir que o tópico SNS envie mensagens para ela. Configure a Política de Acesso da Fila SQS: A parte crucial com os ARNs e a política do SQS.
Volte para o console SQS. Selecione seu fila principal. Vá para a aba Política de filas. Clique em Editar. Exclua a política atual. Copie o arquivo “Política SQS”, descrito abaixo Abra o arquivo em um bloco de notas e faça as alterações devidas:

  >>>>(ARQUIVO COM A POLÍTICA EM ANEXO COM NOME DE JSON)

  ** Onde Colar os ARNs (e ID da Conta):
“AWS”: Cole o seu ID de conta da AWS (12 dígitos). Você encontra o ID
no canto superior direito do console. “Recurso”: “ <COLE AQUI O ARN DA SUA FILA PRINCIPAl> “, cole o ARN completo da sua FILA PRINCIPAL.
“aws:SourceArn”: “ <COLE AQUI O ARN DO SEU TÓPICO SNS> “: Cole
o ARN do seu TÓPICO SNS. Edite com cuidado. Copie e cole a sua política devidamente editada no SQS. Salve.


Parte 4 => Testes

Testar Publicação e Recebimento Normal: Volte ao console SNS e selecione seu tópico. Clique no botão Publicar mensagem. Assunto (Opcional): Teste de Mensagem Normal. Corpo da mensagem:


  >>>>(SCRIPT PARA TESTE EM AXEXO COM NOME TESTE)



Não mexa nos atributos da mensagem por enquanto. Clique em Publicar mensagem. Vá para o console SQS, selecione sua fila principal. Clique em Enviar e receber mensagens. Na seção Receber mensagens, clique em Sondar mensagens. Você deve ver 1 mensagem disponível. Clique nela para ver o corpo (que incluirá a estrutura do SNS se você não marcou “entrega bruta”). IMPORTANTE: Não exclua a mensagem ainda se quiser testar a DLQ no próximo passo. Se você quiser apenas confirmar a coleta, pode excluí-la agora. Testar Envio para a DLQ (Simulação de Falha): Antes de seguirmos, valide as configurações de sondagem de 10 para 3. Acesse sua FILA PRINCIPAL. Desça até “Receber mensagens”. Clique em “Editar configurações de sondagem”. Verifique se você configurou corretamente ao criar o fila. Caso você esteja 3, pode sair sem salvar. Caso esteja 10, altere o “Número máximo de mensagens” para 3. Salvar.

Agora vamos para os testes: Certifique-se de que a mensagem do passo anterior ainda está na fila principal (se a excluiu, publique outra).
Na fila principal, clique em Enviar e receber mensagens. Clique em Pesquisar mensagens. Uma mensagem aparecerá. Observe o Contagem de recebimentos na aba Detalhes da mensagem (deve ser 1).
• Não faça nada com a mensagem. Aguarde o “Tempo limite de visibilidade” da fila expirar (o padrão é 30 segundos). A mensagem voltará
a ficar visível na fila. • Clique em Pesquisar mensagens novamente. A mesma mensagem aparecerá. A Contagem de coletas agora será 2. Repita: aguarde o tempo limite de visibilidade expirar. Clique em Pesquisar mensagens novamente. A Contagem de coleta será 3. Aguarde o tempo limite de visibilidade expirar mais uma vez. Clique em Pesquisar mensagens novamente. A mensagem não deve mais aparecer na fila principal, pois atingiu o “Número máximo de coletas” (3) que configuramos. Agora, vá para a fila DLQ (minha-dlq-lab-…). Clique em Enviar e receber mensagens. Clique em Pesquisar mensagens. A mensagem original que falhou em ser processada na fila principal deve
agora estar na DLQ! Você pode inspecioná-la.

![image](https://github.com/user-attachments/assets/e32ab8c8-c442-461c-b397-fed6502d57c2)


Parte 5 => Limpeza

Limpeza de Recursos (IMPORTANTE para evitar custos):
• Excluir Assinaturas SNS: No console SNS, selecione seu tópico, vá em
“Assinaturas”, selecione a assinatura SQS (e qualquer outra que tenha
criado) e clique em “Excluir”.
• Excluir Tópico SNS: No console SNS, vá em “Tópicos”, selecione seu
tópico e clique em “Excluir”. Confirme a exclusão.
• Excluir Filas SQS: No console SQS, selecione o fila principal e clique
em “Excluir”. Confirme a exclusão. Faça o mesmo para a fila DLQ. Nota:
Às vezes é necessário aguardar um pouco após excluir a assinatura para
conseguir excluir a fila.


=> Laboratório desenvolvido no curso de AWS Developer da Escola da Nuvem

https://www.linkedin.com/school/escola-da-nuvem/

=>Laboratório realizado por Patrícia Sousa

https://medium.com/@patricia--sousa/automatizando-processos-com-aws-6d99f1f3737c

https://www.linkedin.com/posts/patricia--sousa_automatizando-processos-com-aws-activity-7317197423678828544-DIfA?utm_source=share&utm_medium=member_desktop&rcm=ACoAAD9rrFgBvlm-dUoffFLLk0d95JsIQHOe07Y

Aprendizado prático na Nuvem AWS ☁️ | Notificações automáticas e filas resilientes!






