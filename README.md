DataBase config typeORM 

Mais simples para projetos pequenos 

import dotenv from 'dotenv';
import path from 'node:path';
import { DataSourceOptions } from 'typeorm';

dotenv.config();

const baseConfig = {
  entities: [path.join(__dirname, '..', 'entities', '*.{ts,js}')],
  migrations: [path.join(__dirname, '..', 'migrations', '*.{ts,js}')],
  logging: false,
} as const;

const configurations = {
  production: {
    ...baseConfig,
    type: 'postgres',
    host: process.env.DB_HOST,
    port: Number(process.env.DB_PORT),
    username: process.env.DB_USER,
    password: process.env.DB_PASS,
    database: process.env.DB_NAME,
    synchronize: false, // Nunca use true em produção
    ssl: process.env.DB_SSL === 'true',
  },
  test: {
    ...baseConfig,
    type: 'sqlite',
    database: ':memory:', // Banco em memória para testes
    synchronize: true,    // OK para testes
    dropSchema: true,     // Limpa o banco antes dos testes
  },
} as const;

export const getDatabaseConfig = (
  env: keyof typeof configurations
): DataSourceOptions => {
  return configurations[env];
};

dotenv
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASS=senha123
DB_NAME=meu_app
DB_SSL=false


// src/database/AppDataSource.ts
import { DataSource } from 'typeorm';
import { getDatabaseConfig } from '../config/database';

const env = process.env.NODE_ENV === 'test' ? 'test' : 'production';
export const AppDataSource = new DataSource(getDatabaseConfig(env));


#mais complexo e detalhado melhor para projetos grandes

// src/config/database/base.config.ts
import path from 'node:path';
import { DataSourceOptions } from 'typeorm';

export const baseConfig: Partial<DataSourceOptions> = {
  entities: [path.join(__dirname, '..', '..', 'entities', '*.{ts,js}')],
  migrations: [path.join(__dirname, '..', '..', 'migrations', '*.{ts,js}')],
  logging: false,
};

// src/config/database/production.config.ts
import dotenv from 'dotenv';
import { DataSourceOptions } from 'typeorm';
import { baseConfig } from './base.config';

dotenv.config();

export const productionConfig: DataSourceOptions = {
  ...baseConfig,
  type: 'postgres',
  url: process.env.DATABASE_URL, // Para usar com serviços como Heroku
  // Ou configuração detalhada
  host: process.env.DB_HOST,
  port: Number(process.env.DB_PORT),
  username: process.env.DB_USER,
  password: process.env.DB_PASS,
  database: process.env.DB_NAME,
  synchronize: false,
  ssl: {
    rejectUnauthorized: false, // Necessário para alguns provedores
  },
} as const;

// src/config/database/test.config.ts
import { DataSourceOptions } from 'typeorm';
import { baseConfig } from './base.config';

export const testConfig: DataSourceOptions = {
  ...baseConfig,
  type: 'sqlite',
  database: ':memory:',
  synchronize: true,
  dropSchema: true,
  logging: false,
} as const;

// src/config/database/index.ts
import { DataSourceOptions } from 'typeorm';
import { productionConfig } from './production.config';
import { testConfig } from './test.config';

const configurations = {
  production: productionConfig,
  test: testConfig,
} as const;

export const getDatabaseConfig = (
  env: keyof typeof configurations
): DataSourceOptions => {
  return configurations[env];
};



// src/database/AppDataSource.ts
import { DataSource } from 'typeorm';
import { getDatabaseConfig } from '../config/database';

const env = process.env.NODE_ENV === 'test' ? 'test' : 'production';
export const AppDataSource = new DataSource(getDatabaseConfig(env));
