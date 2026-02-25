# TABELA PARA NF-e

## Campos sugeridos:
```
    id: uuid | string | int | Serial
    emitente_id: uuid | string | int - FK tabela do emitente
    numero: string - not null
    serie: string - not null
    payload: json - not null
    chave: string - size: 44
    data_hora_evento: DATETIME | string
    protocolo: string
    status: string - not null
    mensagem: text
    xml: text
    pdf: text
```
OBS: Sugestões de status:

    - Não enviado
    - Em processamento
    - Autorizado
    - Rejeitado
    - Cancelado
    - Denegado
    - Documento Inválido*
    - Erro *
    OBS: campos com * são status opcionais varia da necessidade do cliente.

## Exemplo de Query de criação:
### PostgreSQL:
```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE TABLE nfe (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    emitente_id UUID NOT NULL,
    numero VARCHAR(20) NOT NULL,
    serie VARCHAR(10) NOT NULL,
    payload JSONB NOT NULL,
    chave VARCHAR(44),
    data_hora_evento TIMESTAMP,
    protocolo VARCHAR(50),
    status VARCHAR(30) NOT NULL,
    mensagem TEXT,
    xml TEXT,
    pdf TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
-- Índices importantes
CREATE INDEX idx_nfe_emitente_id ON nfe (emitente_id);
CREATE UNIQUE INDEX idx_nfe_chave ON nfe (chave);
CREATE INDEX idx_nfe_status ON nfe (status);
-- Constraint opcional para evitar duplicidade por número/serie/emitente
CREATE UNIQUE INDEX idx_nfe_emitente_numero_serie
ON nfe (emitente_id, numero, serie);
-- Constraint opcional para status controlado
ALTER TABLE nfe
ADD CONSTRAINT chk_nfe_status
CHECK (status IN (
    'Não enviado',
    'Em processamento',
    'Autorizado',
    'Rejeitado',
    'Cancelado',
    'Denegado'
    'Documento Inválido',
    'Erro'
));
```
### MySQL:
```sql
CREATE TABLE nfe (
    id CHAR(36) PRIMARY KEY,
    emitente_id CHAR(36) NOT NULL,
    numero VARCHAR(20) NOT NULL,
    serie VARCHAR(10) NOT NULL,
    payload JSON NOT NULL,
    chave VARCHAR(44),
    data_hora_evento DATETIME,
    protocolo VARCHAR(50),
    status ENUM(
        'Não enviado',
        'Em processamento',
        'Autorizado',
        'Rejeitado',
        'Cancelado',
        'Denegado',
        'Documento Inválido',
        'Erro'
    ) NOT NULL,
    mensagem LONGTEXT,
    xml LONGTEXT,
    pdf LONGTEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY uk_nfe_chave (chave),
    UNIQUE KEY uk_nfe_emitente_numero_serie (emitente_id, numero, serie),
    KEY idx_nfe_emitente_id (emitente_id),
    KEY idx_nfe_status (status)
);
```