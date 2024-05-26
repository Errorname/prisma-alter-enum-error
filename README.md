# prisma-alter-enum-error

## To reproduce

Update `schema.prisma` with the following diff:

```diff
model User {
  id   String   @id @default(uuid())
  type UserType
}

model Post {
  id         String @id @default(uuid())
+ accessType UserType
}

enum UserType {
  admin
- user
  api
+ something_else
}
```

Then run `npx prisma migrate dev --name failing`

The generated migration content:

```
/*
  Warnings:

  - The values [user] on the enum `UserType` will be removed. If these variants are still used in the database, this will fail.
  - Added the required column `accessLevel` to the `Post` table without a default value. This is not possible if the table is not empty.

*/
-- AlterEnum
BEGIN;
CREATE TYPE "UserType_new" AS ENUM ('admin', 'api', 'something_else');
ALTER TABLE "User" ALTER COLUMN "type" TYPE "UserType_new" USING ("type"::text::"UserType_new");
ALTER TABLE "Post" ALTER COLUMN "accessLevel" TYPE "UserType_new" USING ("accessLevel"::text::"UserType_new");
ALTER TYPE "UserType" RENAME TO "UserType_old";
ALTER TYPE "UserType_new" RENAME TO "UserType";
DROP TYPE "UserType_old";
COMMIT;

-- AlterTable
ALTER TABLE "Post" ADD COLUMN     "accessLevel" "UserType" NOT NULL;
```

This will output the following error:

```
Environment variables loaded from .env
Prisma schema loaded from prisma/schema.prisma
Datasource "db": PostgreSQL database "prisma-alter-enum-error", schema "public" at "localhost:5432"


⚠️  Warnings for the current datasource:

  • The values [user] on the enum `UserType` will be removed. If these variants are still used in the database, this will fail.

✔ Are you sure you want to create and apply this migration? … yes
Applying migration `20240526112721_failing`
Error: ERROR: current transaction is aborted, commands ignored until end of transaction block
   0: schema_core::commands::apply_migrations::Applying migration
           with migration_name="20240526112721_failing"
             at schema-engine/core/src/commands/apply_migrations.rs:91
   1: schema_core::state::ApplyMigrations
             at schema-engine/core/src/state.rs:202
```
