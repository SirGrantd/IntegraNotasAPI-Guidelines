# TABELA PARA NFS-e

## Campos sugeridos:
```
    id: uuid | string | int | Serial
    emitente_id: uuid | string | int - FK tabela do emitente
    numero: string
    numero_rps: string - not null
    serie: string - not null
    payload: json - not null
    chave: string - size: 44
    data_hora_evento: DATETIME | string
    codigo_verificacao: string
    status: string - not null
    mensagem: text
    xml: text
    pdf: text
    pdf_provedor: text
```

OBS: Sugestões de status:

    - Não enviado
    - Em processamento
    - Autorizado
    - Rejeitado
    - Cancelado
    - Documento Inválido*
    - Erro *
    OBS: campos com * são status opcionais varia da necessidade do cliente.

## Exemplo de Query de criação:
### PostgreSQL:
```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE TABLE nfse (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    emitente_id UUID NOT NULL,
    numero VARCHAR(20),
    numero_rps VARCHAR(20) NOT NULL,
    serie VARCHAR(10) NOT NULL,
    payload JSONB NOT NULL,
    chave VARCHAR(44),
    data_hora_evento TIMESTAMP,
    codigo_verificacao VARCHAR(100),
    status VARCHAR(30) NOT NULL,
    mensagem TEXT,
    xml TEXT,
    pdf TEXT,
    pdf_provedor TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
-- Índices
CREATE INDEX idx_nfse_emitente_id ON nfse (emitente_id);
CREATE INDEX idx_nfse_status ON nfse (status);
CREATE UNIQUE INDEX idx_nfse_chave ON nfse (chave);
-- Evita duplicidade de RPS por emitente/série
CREATE UNIQUE INDEX idx_nfse_emitente_rps_serie
ON nfse (emitente_id, numero_rps, serie);
-- Constraint opcional para controle de status
ALTER TABLE nfse
ADD CONSTRAINT chk_nfse_status
CHECK (status IN (
    'Não enviado',
    'Em processamento',
    'Autorizado',
    'Rejeitado',
    'Cancelado',
    'Documento Inválido',
    'Erro'
));
```
### MySQL:
```sql
    CREATE TABLE nfse (
    id CHAR(36) PRIMARY KEY DEFAULT (UUID()),
    emitente_id CHAR(36) NOT NULL,
    numero VARCHAR(20),
    numero_rps VARCHAR(20) NOT NULL,
    serie VARCHAR(10) NOT NULL,
    payload JSON NOT NULL,
    chave VARCHAR(44),
    data_hora_evento DATETIME,
    codigo_verificacao VARCHAR(100),
    status ENUM(
        'Não enviado',
        'Em processamento',
        'Autorizado',
        'Rejeitado',
        'Cancelado',
        'Documento Inválido',
        'Erro'
    ) NOT NULL,
    mensagem LONGTEXT,
    xml LONGTEXT,
    pdf LONGTEXT,
    pdf_provedor LONGTEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY uk_nfse_chave (chave),
    UNIQUE KEY uk_nfse_emitente_rps_serie (emitente_id, numero_rps, serie),
    KEY idx_nfse_emitente_id (emitente_id),
    KEY idx_nfse_status (status)
);
);
```