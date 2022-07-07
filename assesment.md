# Take Home Challenge: Technical Support Engineer

## Request 1: Comparison of Database libraries

**Knex** is a query builder. A query builder is an API that provides functions that can be chained together to form a query. It is an SQL query and schema builder for almost any SQL engine. Knex is more low level than ORMs. it is not aware of relations and it is more verbose. The biggest drawback with SQL query builder is that application developers still need to think about their data in terms of SQL. This incurs a cognitive and practical cost of translating relational data into objects. Another issue is that it's too easy to get overwhelmed if you don't know exactly what you're doing in your SQL queries.

**Sequelize** on the other hand is a popular Node.js ORM library. It is used mostly with SQL engines. It is a traditional ORM which maps tables to model classes. Instances of the model classes then provide an interface for CRUD queries to an application at runtime. Some of the drawbacks include bloated model instances, mixing business with storage logic, lack of type-safety or unpredictable queries caused by lazy loading.

**Prisma** is the easiest to work with when compared to Knex or Sequelize. It has support for both SQL and NOSQL databases. Developer productivity and developer experience is at the core of prisma. It provides type safety which helps reduce typos or mistakes when writing the query. One key feature of Prisma against other ORMs is that it doesn't use object models but rather it uses a schema file to map all the tables and columns. Prisma mitigates many problems associated with traditional ORMs like Sequelize. It uses the Prisma schema to define application models in a declarative way. Prisma Migrate then allows to generate SQL migrations from the Prisma schema and executes them against the database. CRUD queries are provided by Prisma Client, a lightweight and entirely type-safe database client for Node.js and TypeScript.

## Request 2: Nested GraphQL Queries

The `Order` type in the graphQL has one relation. i.e the owner field which denotes a 1-1 relation to User. Also, the `User` type in the graphQL schema has one relation. i.e The orders field which denotes a relation to Order. In order to resolve these relations, you will need to use `type resolvers`. For this specific issue, you will add an `Order` field to your resolver map and implement the resolver for the owner relation as follows:

```js
const resolvers = {
	Query: {
		ordersForUser: (parent, args, context) => {
			return context.prisma.order.findMany({
				where: {
					owner: {
						id: parseInt(args.userId)
					}
				}
			})
		},
		users: (parent, args, context) => {
			return context.prisma.user.findMany()
		}
	},
	Order: {
		owner: (parent, args, context) => {
			return context.prisma.order
				.findUnique({
					where: { id: parent.id },
				}).owner()
		},
	}
}
```

## Request 3: Repository pattern with Prisma

Prisma doesn't have a repository pattern. Prisma is quite different from traditional ORM's in the sense that it doesn't offer any way to inherit or extend methods as there are no classes involved. You directly get the data in the form of a simple JavaScript object and not an instance of any class. What we could do
is if say we have a schema.prisma file like this 
```js
model User {
  id      Int     @id @default(autoincrement())
  name    String
  company Company
}

model Company {
  id     Int    @id @default(autoincrement())
  name   String
  user   User   @relation(fields: [userId], references: [id])
  userId Int
}
```
We could create a function like this 
```js
function queryFromCompany(companyId: number, where: Prisma.UserWhereInput) {
  return prisma.user.findMany({
    where: { company: { id: companyId }, ...where },
  })
}
```
and then use it like this
```js
And then use it like this:
const user = await queryFromCompany(1, { name: 'Paul' })
```

## Request 4: Data validation with Prisma

Prisma Client provides type-safety and runtime type validation but it does not provide validation for user input. It validates input at runtime and throws an error if validation fails i.e the input data does not match the type in the prisma schema. However, we can use any validation  library we like for example, Yup, ZOd, Joi etc.

## Request 5: Can i throw an exception if record doesn't exist in database

It is possible to throw an exception if record does not exist in the database using Prisma. Prior to Prisma v4.0.0, you would use the `rejectOnNotFound` parameter to configure `findUnique` to throw an error if the record is not found. By default, it returns null if the record is not found. To enable an exception to be thrown globally for `findUnique` and `findFirst`, we do
```js
 const prisma = new PrismaClient({
    rejectOnNotFound: true
 })
```
To enable it for a specific operation, we do
```js
  const prisma = new PrismaClient({
    rejectOnNotFound: {
        findUnique: true
    }
  })
```
From version 4.0.0, `findUniqueOrThrow` replaces the `rejectOnNotFound` option. `findUniqueOrThrow`
retrieves a single data record in the same way as `findUnique`. However, if the query does not find
a record, it returns NotFoundError

## Request 6: How to migrate firstname and lastname columns

We can achieve this using expand and contract pattern.

1. Add the new `name` field to the Prisma schema and create a migration.

```js
   model User{
    id Int @id @default (autoincrement)
    firstName String
    lastName  String
    name      String
   }
```

2. Update the application code and write to the `firstName` field, `lastName` field and in the application code, assign the concatenation of both the `firstName` field and `lastName` field to `name` field. At this point, all three fields are being written to.

3. Create an empty migration and copy existing data from `firstName` and `lastName` to the `name` field.

`npx prisma migrate dev --name copy_name --create-only`

```sql
   UPDATE "USER" SET name = concat(firstName, ' ', lastName);
```
4. Verify the integrity of the `name` field in the database.

5. Update application code to read from the new `name` field.

6. Update application code to stop writing to the `firstName` and `lastName` fields.

7. Contract: remove the `firstName` and `lastName` fields from the Prisma schema, and create a migration to remove the `firstName` and `lastName` fields.

## Request 7: Strong language & rude users

#### Message 1
1. Hello [user], I'm Raphael, a Technical Support Engineer with Prisma. I came across your comment on this  github issue [issue name] and would like to thank you for your continued use of the Prisma product. I'm happy you feel excited about our product but i also observed that the language used to convey your excitement does not meet up with our community guidelines as it includes some vulgar slangs. Would you please rephrase the comment using more appropriate words?

2. I would contact them via direct message on slack.

3. I chose slack because i can leverage on our previous connection on slack to sensitize the user
on the acceptable community guidelines.

### Message 2
1. Hello [user], I'm sorry you're having such an experience with the product at this time. Would you
please provide more context on the issue you're currently facing so that we could better support you 
and improve your experience with the product.

2. Threaded Slack response.

3. I chose the threaded Slack response so that it would be easy to keep track of the issue without 
it getting lost in the sea of other messages on slack. It would also serve as a point of reference for the future.
