# Lia Hoff — Comportamento e Bem-estar Animal

Sistema web profissional para **Médica-Veterinária Natalia Martins Hoffmann — CRMV-SC 07725**, com a identidade **Lia Hoff — Comportamento e Bem-estar Animal**.

## V2 — o que foi implementado

- Dashboard e módulos de responsáveis, animais, consultas, agenda, comportamento, evolução, prescrições, financeiro, relatórios e configurações.
- Dados iniciais vazios: **não há pacientes, responsáveis ou valores fictícios**.
- Persistência local automática no navegador (`localStorage`).
- Backup manual em JSON: exportar e importar.
- Indicador de sincronização no topo: Local / Firebase configurado / Sincronizado.
- Camada Firebase preparada para **Firebase Authentication + Cloud Firestore**.
- Edição de responsáveis e animais com formulários pré-preenchidos.
- Arquivamento e restauração de cadastros, mantendo o histórico clínico.
- ID estável por animal e bloqueio de nomes duplicados para evitar prontuários ambíguos.
- Busca global com sugestões de animais e responsáveis.
- Painel com distribuição dos casos por nível de risco.
- Prescrições salvas na lista do sistema e reabertas para impressão/Salvar como PDF.
- Login por e-mail e senha e criação de acesso pelo próprio sistema.
- Sincronização simples por usuário autenticado: `users/{uid}/app/state`.
- Sincronização com revisão para detectar gravações concorrentes; conflitos pausam a sincronização e oferecem usar a nuvem, usar o dispositivo ou mesclar por ID, comparando `updatedAt` no mesmo registro.
- Prescrição separando receita comum de receituários especiais, sem tentar substituir os modelos oficiais.

## Como publicar no GitHub Pages

1. Crie um repositório no GitHub.
2. Envie `index.html` e a pasta `assets`.
3. Abra **Settings → Pages**.
4. Em **Build and deployment**, selecione **Deploy from a branch**.
5. Escolha a branch `main` e a pasta `/ (root)`.
6. Salve e aguarde a publicação.

O `index.html` usa caminhos relativos e pode ser publicado diretamente no GitHub Pages.

## Como ativar o Firebase

### 1. Criar o projeto

No Firebase Console:

1. Crie um projeto para a Lia Hoff.
2. Adicione um **Web App** ao projeto.
3. Copie a configuração web fornecida pelo Firebase.
4. Ative **Authentication → Sign-in method → E-mail/Senha**.
5. Crie o **Cloud Firestore Database**.

A configuração web do Firebase (API key, project ID etc.) é usada pelo navegador; a proteção dos dados depende das **Security Rules** e da autenticação.

### 2. Preencher no sistema

Abra **Configurações → Firebase — sincronização entre dispositivos** e preencha:

- API Key
- Auth Domain
- Project ID
- Storage Bucket
- Messaging Sender ID
- App ID
- E-mail de acesso
- Senha

Clique em **Criar acesso** no primeiro dispositivo ou **Conectar e sincronizar** se a conta já existir.

Depois, no celular/tablet/computador, use a mesma configuração do projeto e faça login com o mesmo acesso.

### 3. Security Rules recomendadas para esta V2

No Firestore, substitua a regra atual por esta e clique em **Publicar**. Ela permite acesso somente aos dois e-mails autorizados e apenas ao espaço identificado pelo UID de cada conta:

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/app/{docId} {
      allow read, write: if request.auth != null
        && request.auth.token.email_verified == true
        && request.auth.token.email in ['liahoff.vet@gmail.com', 'prigeosc@gmail.com']
        && request.auth.uid == userId;
    }
  }
}
```

O formulário do sistema também rejeita outros e-mails e exige confirmação do endereço antes da sincronização. A regra do Firestore é a proteção efetiva dos dados: publique-a no Console e confirme que não há outras regras mais permissivas para esses caminhos. No Firebase Authentication, remova/desative contas que não devam continuar existindo. **Revise as regras antes de usar dados clínicos reais.**

## Como a sincronização funciona nesta V2

- O sistema abre normalmente mesmo sem Firebase.
- Toda alteração é salva localmente.
- Se o Firebase estiver conectado, a alteração também é enviada para o documento do usuário.
- Cada alteração de registro recebe `updatedAt`, usado para escolher a versão mais recente durante uma mesclagem.
- Se a conexão cair, os dados continuam locais; ao voltar a conexão, o sistema compara as versões e pede resolução se houver divergência.
- Ao entrar em outro dispositivo, o sistema recupera o estado salvo na nuvem.
- Se ainda não existir estado na nuvem, o primeiro dispositivo envia o estado local.
- O estado é um documento único nesta fase, adequado para começar e testar a sincronização.

### Limitação consciente

Esta sincronização ainda não foi conectada a um projeto real nem validada entre dois dispositivos. O primeiro teste deve usar dados fictícios e confirmar login, envio, leitura, conflito e restauração de backup. A V2 usa um documento-snapshot único; para uma operação clínica maior, a próxima evolução deve separar os dados em coleções, por exemplo:

- `responsaveis`
- `animais`
- `consultas`
- `casos_comportamentais`
- `triagens`
- `planos_comportamentais`
- `evolucoes`
- `prescricoes`
- `financeiro`
- `auditoria`

Isso permitirá histórico de alterações, permissões mais granulares, consultas eficientes e menor risco de conflito entre dispositivos.

## Módulo de comportamento

A triagem foi estruturada para registrar, entre outros itens:

- queixa principal nas palavras do responsável pelo animal;
- descrição objetiva do comportamento;
- início/mudança;
- frequência e duração;
- contexto e ambiente;
- antecedente → comportamento → consequência;
- gatilhos;
- intensidade;
- risco e sinais de alerta;
- histórico médico e medicações relevantes;
- estratégias já tentadas;
- fatores de bem-estar e necessidades ambientais;
- hipóteses/formulação clínica preenchidas pela profissional.

O sistema **não produz diagnóstico automático, não escolhe medicamentos e não calcula dose**.

## Prescrições e conformidade documental

A aplicação identifica a profissional como **Médica-Veterinária Natalia Martins Hoffmann — CRMV-SC 07725** e usa a marca Lia Hoff.

O módulo diferencia receita comum de tipos sujeitos a regras específicas e mostra um alerta para que receituários especiais sejam emitidos no modelo oficial e conforme o fluxo sanitário vigente. A impressão genérica do sistema não deve ser tratada como substituta automática de Notificação de Receita ou Receita de Controle Especial.

Antes do uso profissional, confira os modelos e requisitos atuais do **CRMV-SC e da Anvisa**, especialmente para medicamentos sujeitos a controle especial e para emissão eletrônica.

## Próxima evolução prevista

1. Transformar o snapshot em coleções Firestore.
2. Criar prontuário completo por animal.
3. Criar plano comportamental com objetivos mensuráveis e evolução.
4. Criar trilha de auditoria.
5. Melhorar agenda e retornos.
6. Criar relatórios profissionais em PDF.
7. Implementar fluxo específico de prescrição eletrônica somente depois de definir o modelo regulatório e a integração necessária.

## Próximas melhorias sugeridas

1. Aplicar permissões por perfil (profissional, recepção e leitura) antes de compartilhar acesso.
2. Criar trilha de auditoria para edição, arquivamento e exclusão, com data e usuário responsável.
3. Migrar as relações clínicas de nomes para IDs em todas as coleções e oferecer uma tela para resolver registros antigos ambíguos.
4. Fazer cópia de segurança automática e tratamento de conflitos de sincronização; hoje o Firebase usa um único snapshot e última gravação vence.
5. Adicionar filtros e indicadores de agenda, retornos e financeiro sem preencher dados demonstrativos.
6. Preparar exportação de prontuário e documentos com controle de acesso e revisão de privacidade antes do uso com dados reais.

## Identificação profissional e empresarial
- **Marca:** Lia Hoff — Comportamento e Bem-estar Animal
- **Profissional:** Médica-Veterinária Natalia Martins Hoffmann
- **CRMV-SC:** 07725
- **Empresa/estabelecimento:** Natalia Hoffmann Serviços Veterinários Ltda - ME
- **CNPJ:** 57.586.835/0001-92

## Próximos passos
- Firebase em produção após revisão de segurança.
- Integração de prontuários, consultas, prescrições e financeiro em coleções individuais.
- Avaliar integração futura com o SNCR conforme documentação oficial e requisitos vigentes.

## Receituário especial — modelos veterinários Anvisa

Ao selecionar **Notificação de Receita A VET** ou **B VET**, o sistema prepara a notificação física no modelo vigente da Anvisa (versão 2), preenchendo proprietário, documento e endereço; identificação do animal; comprador; medicamento; concentração; forma farmacêutica; quantidade; posologia; emitente; data e CRMV. A visualização oferece **Imprimir notificação / salvar PDF**, em página personalizada de 20 × 6 cm.

Confira os dados e a escala antes de imprimir. O documento físico ainda deve cumprir os requisitos de numeração, vias, assinatura e carimbo aplicáveis. O sistema não emite receita eletrônica nem se integra ao SNCR. Produtos veterinários controlados pelo MAPA devem ser emitidos no SIPEAGRO.

Referências oficiais:
- [Anvisa — modelos físicos vigentes](https://www.gov.br/anvisa/pt-br/assuntos/medicamentos/controlados/sncr/receituario-fisico)
- [Anvisa — receituário eletrônico e integração ao SNCR](https://www.gov.br/anvisa/pt-br/assuntos/medicamentos/controlados/sncr/receituario-eletronico)
- [MAPA — emissão pelo SIPEAGRO](https://www.gov.br/agricultura/pt-br/assuntos/insumos-agropecuarios/insumos-pecuarios/produtos-veterinarios/cadastro-de-medicos-veterinarios/quero-emitir-notificacao-de-receita-veterinaria-ou-de-aquisicao-por-medico-veterinario)
