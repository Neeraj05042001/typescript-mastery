## Section 1

**A1.**

- product description: string
- user's account balance: number
- whether an order has been confirmed: boolean
- A variable that starts empty intentionally (no user logged in): null
- A variable declared now but assigned later: undefined

**A2**:
Issues:

- `Number` is capitalized should be `number`.
- `"true"` is wriitten as string should be normal `true` or `false`.
- pincode is wriiten as a number, on pincode no mathematical calculation will be done later so it should alwyas be in string any numberical data which has no mathematical importance should be written as string for example phone number, house address , pincode etc..
- score should be `number` type rather than `unknown`, `unknown` should be used only for the types that are not confiremed that what type it could be so we le it unknown for the ts to infer based on the types of the data being filled to the variable.
- score.tofixed(2)- not sure about this but since it has tyoe unknown so sometimes it can give error when the type is any other than `number` as tofixed() can be applied to the number only.

```typescript
// Correct code
let orderId: number = 1001;
let isActive: boolean = true;
let pincode: number = "110001";
let score: number = 95;
score.toFixed(2); // since it has the type numb er now so it is a valid function applied to the score.
```

**A3:**

```typescript
function dataFroAPI(data){
    if(typeof data.price === number){
        console.log(data.price)
    }else{
        console.log("the price should be a number)
    }
}
```

## Sectio 2

**A4:**

-
-
- A list of allowed HTTP methods that must never be changed at runtime

```typescript
Note: i have written in both format
//  1. A list of city names:
const city: string[] = ["Delhi", "Mumbai", "Bhopal", "Noida"];

const city: Arrays<string> = ["Delhi", "Mumbai", "Bhopal", "Noida"];

// 2. A list of product prices:
const productPrices:number[] = [43. 34, 34, 56, 76]
const productPrices:Arrays<number> = [43. 34, 34, 56, 76]

// 3. An empty array that will hold order IDs (numbers) :
const orderIds:number[] = []
const orderIds:Arrays<number>: []

// 4. A list that can hold either strings or numbers:

const list: (string | number)[] = ["Neeraj", "Amartya", 44, 54, "John", 24, 23, ]

const list:Arrays<(string | Arrays)> = ["Neeraj", "Amartya", 44, 54, "John", 24, 23, ]

// 5. A list of allowed HTTP methods that must never be changed at runtime

 forgot for this i think here readonly type should be used
```

**A5:**

```typescript
const names = ["Delhi", "Mumbai", "Pune"];

const result1 = names.map((n) => n.toUpperCase());
// return: ["DELHI", "MUMBAI", "PUNE"]
// Types infered: string
const result2 = names.filter((n) => n.length > 4);
// return: ["Delhi", "Mumbai, ]
// Types infered:string
const result3 = names.find((n) => n.startsWith("D"));
// return: ["Delhi"]
// Types infered:string
const result4 = names.reduce((acc, n) => acc + n.length, 0);
// return: 15
// Types infered: number
```

**A6:**

corrections: tags should have both string and number type arrays and method should have only array of string type

```typescript
const tags: (string | number)[] = [];
tags.push("typescript");
tags.push(42);

const methods: string[] = ["GET", "POST"];
methods.push("DELETE");
```

**A7:**

```typescript
// 1. A `BlogPost` with a readonly ID, title, content, published status

type BlogPost = {
  readonly id: number;
  title: string;
  content: string;
  published: boolean;
  status: string;
  tags?: string[];
  views?: number;
};

type Author = {
  readonly id: number;
  name: string;
  email: string;
  bio?: string;
};

type BlogPostWithAuthor = BlogPost & {
  author: Author;
};
```

**A8:**

- Fixes:
- ProductId - `P` is capital should be small to maintain consistency
- qunatity type is `Number` should be `number`
- in variable decalration discoundt is assigned as string which will throw an error by ts must be a number and also notes is assigned in the variable but not defined in types, either remove from variable or add a type in type alias. i am adding in type aliaas
-

```typescript
type orderItem = {
  productId: number;
  productName: string;
  quantity: number;
  unitPrice: number;
  discount?: number;
  notes?: string;
};

const item: orderItem = {
  productId: 1,
  productName: "Laptop",
  quantity: 2,
  unitPrice: 50000,
  discount: 10,
  notes: "Gift wrap please",
};
```

**A9:**

```typescript
type DeliveryAddress = {
  houseNo: string;
  block: string | number;
  nearby?: string;
  pin: number;
  phone: number;
};

type CustomerOrder = {
  readonly id: number;
  name: string;
  address: DeliveryAddress;
  total: number;
  status: "pending" | "processing" | "delivered";
  notes?: string;
};
```

**10:**

```typescript
// 1. A function that takes an array of strings and returns them sorted alphabetically

function sortedAlphabetically(s: string[]): string[] {
  return s.sort();
}
sortedAlphabetically(["Banana", "Apple", "Orange", "Mango"]);

// 2. A function that takes a price and an optional discount percentage,and returns the final price (default discount of 0)

function finalPrice(price: number, discount?: number): number {
  if (discount) {
    return `The final price is ${price - (discount / 100) * price}`;
  }

  return `The final price is ${price}`;
}

// 3. A function that takes a user ID and always throws "User not found" as an error

function user(userId: number) {
  if (!userId) throw new Error("User not found");
}

// 4. An async function that takes an order ID (number) and returns a `Promise` that resolves to an `{ id: number; status: string }` object

type User = {
  id: number;
  status: string;
};

async function promise(orderId: number): Promise<User> {
  return {
    id: orderId,
    status: "processing",
  };
}
```

**A11:**

```typescript
// 1. A function that takes a string and returns a string (a transformer)

type Transformer = (a: string) => string;

// 2. A function that takes two numbers and returns a boolean(a comparator)

type booleanFunction = (a: number, b: number) => boolean;

// 3. An async function that takes a number ID and returns `Promise<void>`

type promiseFunction = (a: number) => Promise<void>;
```

**A12:**

Bugs:

- in the createorder function the optional type is incorrect customerid should be a mandatory field and note can be the optional field.
- in the applyDiscount function the return type of the function is wron since the return statement is a mathematical calculation so the return type of the function must be a number and not a string.

- in logOrder function the return type of the function is wrong it should be a number as it is consoling an orderid which is a number so it must be a number type instead of type never. never should be used only for the functions which will never return anything.

```typescript
function createOrder(
  customerId: number,
  productId: number,
  quantity: number,
  notes?: string,
): object {
  return { customerId, productId, quantity, notes };
}

function applyDiscount(price: number, discount: number): number {
  return price - (price * discount) / 100;
}

function logOrder(orderId: number): number {
  console.log(`Processing order ${orderId}`);
}
```

**A13:**

```typescript
function requireAdmin(req: Request, res: Response, next): void {
  if (req.body.role !== "Admin") {
    res.status(403).send({ message: "Forbidden" });
  }
  next();
}
```

## Section 5

**A14:**

```typescript
interface BaseEntity {
  readonly id: number;
  readonly createdAt: Date;
  readonly updatedAt: Date;
}

interface Product extends BaseEntity {
  name: string;
  price: number;
  category: string;
  description?: string;
}

interface DigitalProduct extends Product {
  downloadUrl: string;
  fileSizeInMb: string;
}
```

**A15:**

- When using interface type declaration, so the same type name can be redecalred again and again and the variable type decalration gets updated with the latest type declaration but in type alias the redecalaration or the duplication of the same name of the type variable is not allowed. it is so to reduce the conflicts such that when working on a project one developer changes the type to a new one while earlier was set to something so it can break the types and can cause error so type alias are made not to be redecalred to avoid these types of conflicts.

- Express's `Request` type: when the request is assigned the `req` type then ts assigns all the methods of the request to it and suggests and if there any typo or any non-extentet method then it flags the error

```typescript

const user(req:Request, res:Response, next:NextFunction):void{
    if(req.body.user){
        console.log(req.parms) //here it will flag the error as the parms does not exist did you mean params.
        res.send(`The user name is ${req.body.user.name}`)
    }
}

```

**A16:**
Bugs:

- in interface "=" is not used. wrong syntax
- & is also not the part of interface syntax
- {} empty

```typescript
interface BlogComment {
  id: number;
  content: string;
  likes: number;
}

interface FeaturedComment extends BlogComment { isFeatured: boolean }

const comment: FeaturedComment = {
  id: 1,
  content: "Great post"
  likes: 10,
  isFeatured: true
}
```

**A17:**

```ts
type Category = "electronics" | "clothing" | "books";

interface Product extends Category {
  id: number;
  name: string;
  price: number;
  category: string;
  rating?: number;
}

type ProductListProps = {
  products: Product[];
  onSelect: (p: Product) => void;
  isLoading?: boolean;
};

function ProductList(product: Product): ProductListProps {
  return {
    //    JSX.................
  };
}
```

**A18:**

```ts
type CreateBookBody = {
  title: string;
  author: string;
  price: number;
  isbn: string;
  genre?: string;
};

type BookResponse = CreateBookBody & {
  readonly id: number;
  readonly createdAt: Date;
};

app.post("/books", (req: Request, res: Response<BookResponse>): void => {
  const book: BookResponse = {
    title: "Zero to 1",
    author: "Neeraj",
    price: 500,
    isbn: "sbdsjdnbk",
    genre: "self-help",
    id: 5,
    createdAt: Date.now(),
  };

  res.status(200).json(book);
});
```

**A19:**

- A union of three possible user roles:
  **A:** A union of three possible user roles should be declared with the `type alias` as because type alias prevent reassignment further anywhere, means roles can be changes inly from here not anywheher else while in interface it can be reassigned anywhere and also the union is allowed in `type alias` only
- The shape of a database entity that other entities will inherit from
  **A: ** Here both can be used `type alias` as well as `interface` both allows this inheritance using `&` in type alias and `extends` in interface
- A function callback type for an event handler
  **A:** for an event handler its better to use `type alias` as it is best for function signatures and use `interface` only for the object shape.
- A shape that might be augmented by a third-party library later
  **A:** use `interface` for this as the shape can be an object and here the type should be flexible as a third party library data shape can be anything

- A tuple representing latitude and longitude
  **A:** Tuples should alwyas be prefered to use with `type alias`

**A20:**

```ts
interface Config {
  apiUrl: string;
  timeout: number;
  readonly version: string;
}

function createConfig(apiUrl: string, timeOut: number): Config {
  const response = fetch(apiUrl);
  if (!response.ok) throw new Error("Data not found");
  const data: Config = response.json();
  return { data };
}

createConfig("https://api.example.com", 5000);
```
