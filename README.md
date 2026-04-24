# Firebase React Tools

A simple library to create projects fast and scalable with **Firebase V9** and **React**.

[![npm version](https://img.shields.io/npm/v/firebase-react-tools)](https://www.npmjs.com/package/firebase-react-tools)
[![license](https://img.shields.io/npm/l/firebase-react-tools)](https://github.com/freddywebmaster/firebase-react-tools/blob/main/LICENSE)

---

## Table of Contents

- [Installation](#installation)
- [1. Configuration](#1-configuration)
- [2. Authentication](#2-authentication)
  - [AuthService](#authservice)
  - [useAuth hook](#useauth-hook)
- [3. Firestore](#3-firestore)
  - [FirestoreService](#firestoreservice)
  - [useDocument hook](#usedocument-hook)
  - [useQuery hook](#usequery-hook)
  - [useDocumentRT hook (Real-Time)](#usedocumentrt-hook-real-time)
  - [useQueryRT hook (Real-Time)](#usequeryrt-hook-real-time)
  - [useMutation hook](#usemutation-hook)
- [4. Storage](#4-storage)
  - [useStorage hook](#usestorage-hook)
- [TypeScript Types](#typescript-types)

---

## Installation

```bash
npm install firebase-react-tools
```

> No need to install `firebase` separately — the library bundles its own version to avoid dependency conflicts.

---

## 1. Configuration

Initialize Firebase once and export your services to reuse them throughout your app.

```typescript
import {
  AuthService,
  FirestoreService,
  initFirebaseTools,
} from "firebase-react-tools";
import { User, Company, CompanyPermission } from "./interfaces";

const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "your-app.firebaseapp.com",
  projectId: "your-app",
  storageBucket: "your-app.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID",
};

// Initialize the Firebase app with an optional name (useful for multiple apps)
export const myApp = initFirebaseTools(firebaseConfig, "my-app");

// Auth service
export const myAuth = new AuthService(myApp);

// Firestore collections — typed with your interfaces
export const myDb = {
  users: new FirestoreService<User>(myApp, "users"),
  companies: new FirestoreService<Company>(myApp, "companies"),
  companyPermissions: new FirestoreService<CompanyPermission>(myApp, "company-permissions"),
};
```

### `initFirebaseTools(credentials, nameApp?)`

| Parameter     | Type              | Required | Description                                      |
|---------------|-------------------|----------|--------------------------------------------------|
| `credentials` | `FirebaseOptions` | Yes      | Your Firebase project configuration object       |
| `nameApp`     | `string`          | No       | Name for the app instance (default: `[DEFAULT]`) |

Returns a `FirebaseApp` instance.

---

## 2. Authentication

### AuthService

Instantiate once and export it. All methods return a promise.

```typescript
import { AuthService, initFirebaseTools } from "firebase-react-tools";

const app = initFirebaseTools(firebaseConfig, "my-app");
export const myAuth = new AuthService(app);
```

#### Methods

##### `loginEmailAccount(email, password)`
```typescript
const res = await myAuth.loginEmailAccount(email, password);
// res: { error: boolean, message: string, user?: User | null }

if (!res.error) {
  console.log("Logged in:", res.user);
}
```

##### `createEmailAccount(email, password, name?)`
```typescript
const res = await myAuth.createEmailAccount(email, password, "John Doe");

if (res.error) {
  console.error(res.message);
}
```

##### Social Login
```typescript
const res = await myAuth.loginGoogle();
const res = await myAuth.loginGithub();
const res = await myAuth.loginFacebook();
const res = await myAuth.loginTwitter();
```

##### `closeSession(callback?)`
```typescript
await myAuth.closeSession();
// or with callback
await myAuth.closeSession(() => router.push("/login"));
```

##### `sendResetPassword(email, callback?)`
```typescript
await myAuth.sendResetPassword("user@email.com", () => {
  console.log("Reset email sent");
});
```

##### `updatePass(password, newPassword, callback?)`
```typescript
await myAuth.updatePass("oldPass123", "newPass456", () => {
  console.log("Password updated");
});
```

##### `UpdateProfile(data, callback?)`
```typescript
await myAuth.UpdateProfile({
  displayName: "New Name",
  photoURL: "https://example.com/photo.jpg",
});
```

##### `updateEmail(password, newEmail, callback?)`
```typescript
await myAuth.updateEmail("currentPass", "new@email.com");
```

##### `sendVerification(callback?)`
```typescript
await myAuth.sendVerification(() => {
  console.log("Verification email sent");
});
```

##### `deleteAccount(password, callback?)`
```typescript
await myAuth.deleteAccount("userPassword");
```

##### `reAuthUser(password, callback)`
```typescript
await myAuth.reAuthUser("userPassword", () => {
  console.log("Re-authenticated");
});
```

##### `currentUser()`
```typescript
const user = myAuth.currentUser(); // returns User | null
```

#### AuthService Method Reference

| Method                | Parameters                                    | Returns                   |
|-----------------------|-----------------------------------------------|---------------------------|
| `loginEmailAccount`   | `email: string, password: string`             | `Promise<IAuthResponse>`  |
| `createEmailAccount`  | `email: string, password: string, name?: string` | `Promise<IAuthResponse>` |
| `loginGoogle`         | —                                             | `Promise<IAuthResponse>`  |
| `loginGithub`         | —                                             | `Promise<IAuthResponse>`  |
| `loginFacebook`       | —                                             | `Promise<IAuthResponse>`  |
| `loginTwitter`        | —                                             | `Promise<IAuthResponse>`  |
| `closeSession`        | `callback?: Function`                         | `Promise<void>`           |
| `sendResetPassword`   | `email: string, callback?: Function`          | `Promise<void>`           |
| `updatePass`          | `password: string, newPassword: string, callback?: Function` | `Promise<void>` |
| `UpdateProfile`       | `data: IUpdateProfile, callback?: Function`   | `Promise<void>`           |
| `updateEmail`         | `password: string, newEmail: string, callback?: Function` | `Promise<void>` |
| `sendVerification`    | `callback?: Function`                         | `Promise<void>`           |
| `deleteAccount`       | `password: string, callback?: Function`       | `Promise<void>`           |
| `reAuthUser`          | `password: string, callback: Function`        | `Promise<void>`           |
| `currentUser`         | —                                             | `User \| null`            |

---

### useAuth hook

Listens to Firebase auth state and returns the current user with a loading flag.

```typescript
import { useAuth } from "firebase-react-tools";
import { myApp } from "./firebase"; // your initialized app

function App() {
  const { user, loading } = useAuth(myApp);

  if (loading) return <p>Loading...</p>;
  if (!user) return <p>Not logged in</p>;

  return <p>Hello, {user.displayName}</p>;
}
```

#### Return value

| Property  | Type           | Description                           |
|-----------|----------------|---------------------------------------|
| `user`    | `User \| null` | Current Firebase auth user            |
| `loading` | `boolean`      | `true` while auth state is resolving  |

---

## 3. Firestore

### FirestoreService

A typed service for a single Firestore collection.

```typescript
import { FirestoreService } from "firebase-react-tools";

export const usersDb = new FirestoreService<User>(app, "users");
```

#### Constructor

```typescript
new FirestoreService<T>(app: FirebaseApp, collection: string)
```

#### Methods

##### `add(data, id?)`

Add a document. Pass an `id` to set a specific document ID; omit it to auto-generate.

```typescript
// Auto-generated ID
await myDb.companies.add({ name: "Acme", active: true });

// Custom ID
await myDb.companies.add({ name: "Acme" }, "custom-id-123");
```

##### `findById(id)`
```typescript
const res = await myDb.users.findById("user-id-123");

if (!res.error) {
  console.log(res.data); // typed as User
}
```

##### `find(queryOptions?)`

Fetch all documents or filter with Firestore query constraints.

```typescript
import { where, orderBy, limit } from "firebase/firestore";

// All documents
const res = await myDb.users.find();

// With filters
const res = await myDb.users.find(where("active", "==", true));
```

##### `update(id, newData, merge?)`

Update a document. `merge` defaults to `true` (partial update).

```typescript
await myDb.companies.update("company-id", { name: "New Name" });

// Full overwrite
await myDb.companies.update("company-id", { name: "New Name" }, false);
```

##### `delete(id)`
```typescript
const res = await myDb.users.delete("user-id-123");
```

##### `transaction(id, field, value)`

Atomically increment or decrement a numeric field.

```typescript
// Increment "views" by 1
await myDb.posts.transaction("post-id", "views", 1);

// Decrement "stock" by 1
await myDb.products.transaction("product-id", "stock", -1);
```

##### `addInArray(id, field, data)`

Add an item to an array field.

```typescript
await myDb.companies.addInArray("company-id", "roles", { name: "admin", permissions: [] });
```

##### `deleteInArray(id, field, data)`

Remove an item from an array field.

```typescript
await myDb.companies.deleteInArray("company-id", "roles", roleObject);
```

##### `deleteField(id, field)`

Delete a specific field from a document.

```typescript
await myDb.users.deleteField("user-id", "temporaryToken");
```

##### `documentSuscribe(id, callback)`

Real-time listener for a single document. Returns an unsubscribe function.

```typescript
const unsub = myDb.users.documentSuscribe("user-id", (snapshot) => {
  console.log(snapshot.data());
});

// Cleanup
unsub();
```

##### `collectionSuscribe(callback, queryOptions?)`

Real-time listener for a collection. Returns an unsubscribe function.

```typescript
const unsub = myDb.users.collectionSuscribe((users) => {
  console.log(users); // typed as User[]
});

// With filter
const unsub = myDb.users.collectionSuscribe(
  (users) => console.log(users),
  where("active", "==", true)
);

// Cleanup
unsub();
```

#### FirestoreService Method Reference

| Method              | Parameters                                                        | Returns                                    |
|---------------------|-------------------------------------------------------------------|--------------------------------------------|
| `add`               | `data: DocumentData, id?: string`                                 | `Promise<IResponseFirestore<T>>`           |
| `findById`          | `id: string`                                                      | `Promise<IResponseFirestore<T>>`           |
| `find`              | `queryOptions?: QueryConstraint`                                  | `Promise<IResponseFirestore<T[]>>`         |
| `update`            | `id: string, newData: DocumentData, merge?: boolean`              | `Promise<IResponseFirestore<T>>`           |
| `delete`            | `id: string`                                                      | `Promise<IResponseFirestore<string>>`      |
| `transaction`       | `id: string, field: string, value: number`                        | `Promise<IResponseFirestore<T>>`           |
| `addInArray`        | `id: string, field: string, data: any`                            | `Promise<IResponseFirestore<{error, message}>>`   |
| `deleteInArray`     | `id: string, field: string, data: any`                            | `Promise<IResponseFirestore<{error, message}>>`   |
| `deleteField`       | `id: string, field: string`                                       | `Promise<IResponseFirestore<{error, message}>>`   |
| `documentSuscribe`  | `id: string, callBack: (doc: DocumentSnapshot) => void`           | `Unsubscribe`                              |
| `collectionSuscribe`| `callBack: (collection: T[]) => void, queryOptions?: QueryConstraint` | `Unsubscribe`                          |

---

### useDocument hook

Fetches a single Firestore document and provides a `refetch` function. Optionally uses a cache while loading.

```typescript
import { useDocument } from "firebase-react-tools";
import { myDb } from "./firebase";

function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);

  const { isLoading, error, refetch, mutate } = useDocument(
    myDb.users,  // FirestoreService<User>
    setUser,     // state setter
    userId,      // document ID
    user ?? undefined // optional cache (shows stale data while refetching)
  );

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error loading user</p>;

  return <p>{user?.displayName}</p>;
}
```

#### Parameters

| Parameter     | Type                          | Required | Description                              |
|---------------|-------------------------------|----------|------------------------------------------|
| `service`     | `FirestoreService<T>`         | Yes      | The Firestore service for the collection |
| `setState`    | `(data: T) => void`           | Yes      | State setter function                    |
| `id`          | `string`                      | Yes      | Document ID to fetch                     |
| `objectCache` | `T`                           | No       | Cached data to use while refetching      |

#### Return value

| Property    | Type                    | Description                          |
|-------------|-------------------------|--------------------------------------|
| `isLoading` | `boolean`               | `true` while fetching                |
| `error`     | `boolean`               | `true` if the fetch failed           |
| `refetch`   | `() => Promise<void>`   | Manually re-fetch the document       |
| `mutate`    | `FirestoreService<T>`   | The service for direct mutations     |

---

### useQuery hook

Fetches a Firestore collection with optional query constraints.

```typescript
import { useQuery } from "firebase-react-tools";
import { where } from "firebase/firestore";
import { myDb } from "./firebase";

function CompanyList() {
  const [companies, setCompanies] = useState<Company[]>([]);

  const { isLoading, error, refetch } = useQuery(
    myDb.companies,
    setCompanies,
    { queryOptions: where("active", "==", true) },
    companies // optional cache
  );

  if (isLoading) return <p>Loading...</p>;

  return (
    <ul>
      {companies.map((c) => <li key={c.id}>{c.name}</li>)}
    </ul>
  );
}
```

#### Parameters

| Parameter      | Type                                         | Required | Description                              |
|----------------|----------------------------------------------|----------|------------------------------------------|
| `service`      | `FirestoreService<T>`                        | Yes      | The Firestore service for the collection |
| `setState`     | `(data: T[]) => void`                        | Yes      | State setter function                    |
| `options`      | `{ queryOptions?: QueryConstraint }`         | No       | Firestore query constraints              |
| `arrayCache`   | `T[]`                                        | No       | Cached array to use while refetching     |

#### Return value

| Property    | Type                    | Description                          |
|-------------|-------------------------|--------------------------------------|
| `isLoading` | `boolean`               | `true` while fetching                |
| `error`     | `boolean`               | `true` if the fetch failed           |
| `refetch`   | `() => Promise<void>`   | Manually re-fetch the collection     |
| `mutate`    | `FirestoreService<T>`   | The service for direct mutations     |

---

### useDocumentRT hook (Real-Time)

Subscribes to a Firestore document and calls a callback on every update.

```typescript
import { useDocumentRT } from "firebase-react-tools";
import { myDb } from "./firebase";

function LiveUser({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);

  useDocumentRT(myDb.users, userId, (updatedUser) => {
    setUser(updatedUser);
  });

  return <p>{user?.displayName}</p>;
}
```

#### Parameters

| Parameter | Type                      | Required | Description                              |
|-----------|---------------------------|----------|------------------------------------------|
| `service` | `FirestoreService<T>`     | Yes      | The Firestore service for the collection |
| `id`      | `string`                  | Yes      | Document ID to subscribe to              |
| `cb`      | `(data: T) => void`       | Yes      | Callback called on every update          |

---

### useQueryRT hook (Real-Time)

Subscribes to a Firestore collection and calls a callback on every update.

```typescript
import { useQueryRT } from "firebase-react-tools";
import { where } from "firebase/firestore";
import { myDb } from "./firebase";

function LiveCompanyList() {
  const [companies, setCompanies] = useState<Company[]>([]);

  useQueryRT(
    myDb.companies,
    (updatedCompanies) => setCompanies(updatedCompanies),
    where("active", "==", true)
  );

  return (
    <ul>
      {companies.map((c) => <li key={c.id}>{c.name}</li>)}
    </ul>
  );
}
```

#### Parameters

| Parameter      | Type                        | Required | Description                              |
|----------------|-----------------------------|----------|------------------------------------------|
| `service`      | `FirestoreService<T>`       | Yes      | The Firestore service for the collection |
| `cb`           | `(data: T[]) => any`        | Yes      | Callback called on every update          |
| `queryOptions` | `QueryConstraint`           | No       | Firestore query constraints              |

---

### useMutation hook

Generic hook for wrapping any async function with a loading state.

```typescript
import { useMutation } from "firebase-react-tools";

function DeleteButton({ userId }: { userId: string }) {
  const { isLoading, initMutation } = useMutation(() =>
    myDb.users.delete(userId)
  );

  return (
    <button onClick={initMutation} disabled={isLoading}>
      {isLoading ? "Deleting..." : "Delete"}
    </button>
  );
}
```

#### Parameters

| Parameter | Type                 | Required | Description                     |
|-----------|----------------------|----------|---------------------------------|
| `fn`      | `() => Promise<T>`   | Yes      | Async function to execute       |

#### Return value

| Property      | Type                  | Description                          |
|---------------|-----------------------|--------------------------------------|
| `isLoading`   | `boolean`             | `true` while the function is running |
| `initMutation`| `() => Promise<void>` | Call this to trigger the function    |

---

## 4. Storage

### useStorage hook

Upload files to Firebase Storage and optionally persist metadata to Firestore. Also supports file deletion by URL.

```typescript
import { useStorage } from "firebase-react-tools";
import { myApp } from "./firebase";

function UploadAvatar() {
  const { uploading, uploadFile, deleteImage } = useStorage(myApp);

  const handleUpload = async (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (!file) return;

    const res = await uploadFile("avatars/user-123", file);

    if (!res.error) {
      console.log("URL:", res.file?.url);
      console.log("Size:", res.file?.metadata?.size);
    }
  };

  const handleDelete = async () => {
    await deleteImage("https://firebasestorage.googleapis.com/...");
  };

  return (
    <div>
      <input type="file" onChange={handleUpload} disabled={uploading} />
      {uploading && <p>Uploading...</p>}
    </div>
  );
}
```

#### `uploadFile(reference, file, saveInDb?)`

| Parameter   | Type      | Required | Description                                        |
|-------------|-----------|----------|----------------------------------------------------|
| `reference` | `string`  | Yes      | Storage path (e.g. `"avatars/user-123"`)           |
| `file`      | `File`    | Yes      | File to upload                                     |
| `saveInDb`  | `boolean` | No       | Whether to persist file metadata to Firestore      |

Returns `Promise<iResponseStorage>`:
```typescript
{
  error: boolean;
  file?: {
    url: string;
    metadata?: {
      contentType: string | undefined;
      fullPath: string;
      size: number;
      name: string;
    };
  };
}
```

#### `uploadFiles(reference, files, saveInDb?)`

Upload multiple files at once.

```typescript
const handleMultiUpload = async (e: React.ChangeEvent<HTMLInputElement>) => {
  if (!e.target.files) return;

  const results = await uploadFiles("gallery/user-123", e.target.files);
  const urls = results.filter((r) => !r.error).map((r) => r.file?.url);
};
```

#### `deleteImage(imageUrl)`

Delete a file from Storage using its full download URL.

```typescript
await deleteImage("https://firebasestorage.googleapis.com/v0/b/...");
```

#### useStorage Return value

| Property      | Type                                                      | Description                          |
|---------------|-----------------------------------------------------------|--------------------------------------|
| `uploading`   | `boolean`                                                 | `true` while any upload is in progress |
| `uploadFile`  | `(ref: string, file: File, saveInDb?: boolean) => Promise<iResponseStorage>` | Upload a single file |
| `uploadFiles` | `(ref: string, files: FileList, saveInDb?: boolean) => Promise<iResponseStorage[]>` | Upload multiple files |
| `deleteImage` | `(imageUrl: string) => Promise<void>`                     | Delete file by download URL          |

---

## TypeScript Types

```typescript
// Auth
interface IAuthResponse {
  error: boolean;
  message: string;
  user?: User | null;
}

interface IUpdateProfile {
  displayName?: string;
  photoURL?: string;
}

// Firestore
interface IResponseFirestore<T> {
  error: boolean;
  message: string;
  data?: T;
}

// Storage
interface iResponseStorage {
  error: boolean;
  file?: {
    url: string;
    metadata?: {
      contentType: string | undefined;
      fullPath: string;
      size: number;
      name: string;
    };
  };
}
```

---

## License

MIT © [fwebmaster](https://github.com/fwebmaster-gt)
