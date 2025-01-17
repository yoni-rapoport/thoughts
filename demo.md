---
marp: true
paginate: true

---
# Create a full stack app using react, express & remult
---
# Prepare
<style>
  pre {
  line-height: 150%;
}

blockquote {
  background-color: black;
  color: white;
  padding:1em;
  font-family: "Arial";
  font-size:48px
}
</style>

- activate zen mode
- drop task table on pg
- delete COOKIES
- make sure network tab has only method and is correctly scaled
- Size the terminal window
- open command and scale it to size
- fix bash `notepad c:\Program Files\git\etc\profile.d\git-prompt.sh`

---
> Hi I'm Noam honig, and in this video we'll write a full stack Todo app from scratch - using TypeScript, React, a NodeJS-express API server, and Remult.

> By The end of this video we'll have a working Todo App, that uses a Postgres database, and has basic authentication, and we'll deploy it to the cloud.

---


> We'll start by using vite to create our react app

```sh
npm create vite todo-demo -- --template react-ts
```
```sh
cd todo-demo
npm i 
code . // open vs code
```
open terminal
```sh
npm run dev
```
open vite in browser.

---
# We'll start by creating a basic todo list in react
1. Clear app.tsx
2. Delete app.css
3. delete contents of index.css

> The fist thing we need is a model for the todo items - we'll call it the Task model

4. add `shared/Task.ts`

---
```ts
function App() {
  const [tasks, setTasks] = useState<Task[]>([
    {id:1, title:"Setup", completed:false }
  ]);
  return <>
    <h1>Todos</h1>
    <main>
      <ul>
        {tasks.map(task => {
          return <li key={task.id}>
            {task.title}
          </li>
        })}
      </ul>
    </main>
  </>
}
```

> let's add a checkbox that indicates the "completed" state of each task

```html
<input type="checkbox" checked={task.completed}/>
```
---
# Now let's add a backend
install stuff

> We need Express to serve our app's API, and Remult to do... well... you'll see in a minute...

```sh
npm i express remult
```

> For development, we'll use ts-node-dev to run the API server in watch mode.

```sh
npm i -D @types/express ts-node-dev
```

---

# configuration
> Our server Node.js project is using the CommonJS module system. So we have to remove the "type": "module" entry from the package.json file that was created by Vite.

1. `package.json` remove `type`: `module` -  For our node js server, we'll use common js, so we need to remove type-module from the package json***
   
---

2. Create `tsconfig.server.json`
```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "module": "commonjs",
    "esModuleInterop": true
  }
}
```

---
3. Edit `tsconfig.json`, under `compilerOptions` remove `"experimentalDecorators": true`

> We recommend using Remult decorators, but if you prefer not using decorators, or using javascript instead of typescript, that's OK, head on to remult's documentation and you'll find out how to do that.

---

# Create the Node JS server

1. create server express `src/server/index.ts`
   ```ts
   import express from "express";
   const app = express();
   app.listen(3002, () => console.log("Started"));
   ```
2. Add script to package json
   ```json
   "dev-node":"ts-node-dev -P tsconfig.server.json src/server",
   ```
---
3. Open a new terminal & start the node server
   ```sh
   npm run dev-node
   ```
4. create `api.ts` file
5. register it on the server

---
# Entity

  - Add the Entity decorators
> The Entity decorator tells Remult that this class will be used as a model for both frontend and backend code, so Remult should create CRUD API endpoints and a database table for this entity. 

---
[ highlight allow api crud ]
>  I’ve told Remult to allow all CRUD operations for the Task entity. Later, I’ll restrict that.


[add field decorators]
> We also have decorators for each field to define its datatype and behavior.

  - Indicate that we support also uuid
---
- Register to API
- Demo get
- Demo post:
  ```json
  { "title":"Setup","completed":true }
  ```
  - Entities
  - Paging, Sorting & Filtering
- Explain that data is saved by default to a json db, and we fully support all databases including ....


---
# Now let's use it in the frontend

> Remember that, in development, our API is served from port 3002, while our React app is serves from port 5173. So, in order to access our API from the React app, we have two options - either configure CORS, or use the proxy feature of Vite to forward API calls to port 3002. 
---
> Let's configure the proxy in vite.config.ts to forward all requests sent to routes beginning with `/api` - to the backend server that is listening on port 3002.

3. Configure proxy `vite.config.ts`
   ```ts
   server: { proxy: { '/api': 'http://localhost:3002' } }
   ```

4. Demo proxy

---

# use remult to get the data from the server
```tsx
const taskRepo = remult.repo(Task);
```
> we ask Remult to provide us with a repository object for the Task entity. We use this taskRepo object to interact with the backend.

```tsx
useEffect(()=>{
  taskRepo.find().then(setTasks)
},[])
```
--- 
> You can see the GET request generated by this “find” call over here.
---



>Let’s have a closer look at what we can do with this “find” method up here. Right now it just returns all the tasks from the database, but we can easily use it for paging
1. Show limit and page
   1. Show network tab
---
>These definitions go through the REST API request and into the database query on the backend.

2. Show order by
3. Show Where
> you can see that everything here is simple REST API requests
---
> Remember all I had to do was write a simple TypeScript class and register it with Remult, and I get a type-safe API client, with full auto-complete, and an API endpoint that fetches data from a database table, that was also created for me by Remult. Nice!
---

# Add new Task
> Let's add the "Add Task" functionality to our app
```ts
const [newTask, setNewTask] = useState("");
```
```html
<input placeholder="What needs to be done"
  value={newTask}
  onChange={e => setNewTask(e.target.value)}
  onKeyDown={e => {
    if (e.key === "Enter")
      addTask()
  }} />
```
---
```ts
const addTask = async () => {
  const task = await taskRepo.insert({
    title:newTask
  })
  setTasks([...tasks, task])
  setNewTask('');
}
```


---


# Add a few tasks

> Let's complete the list of features for this todo app

---

- Create 
- Update 
- Delete 
- Validation
- Backend methods
- Authentication
- Authorization
- Database
- Deployment
---
# Set Completed
```ts
const setCompleted = async (completed: boolean) => {
  const updatedTask = await taskRepo.save({ ...task, completed });
  setTasks(tasks.map(t => t === task ? updatedTask : t));
}
```
```tsx
onChange={e => setCompleted(e.target.checked)}
```
---

# Edit Task

```ts
const setTitle = async (title: string) => {
  const t = { ...task, title };
  // refactor replace task
  replaceTask(t);
}
```
```ts
const replaceTask = (theTask: Task) => {
  setTasks(tasks.map(t => t === task ? theTask : t));
}
```
```html
<input
  value={task.title}
  onChange={e => setTitle(e.target.value)} 
  />
```
--- 

# Save Task Edit
```ts
const saveTask = async () => {
  replaceTask(await taskRepo.save(task));
}
```
```ts
onBlur={saveTask}
```
---
# Delete a task
```ts
const deleteTask = async () => {
  await taskRepo.delete(task);
  setTasks(tasks.filter(t => t !== task));
}
```
```tsx
<button onClick={deleteTask} >x</button>
```

--- 
>Did you notice I didn't write any backend code to get the rest of these CRUD operations to work? That's because, with Remult, writing simple full-stack CRUD is, well... simple!
---
# Validation - `Task.ts`

> Now let’s see how, with Remult, I can add data validation to the Task entity, and have it affect both the frontend and the backend.

--- 
> So let’s go over to our Task entity and let’s send an “options” object to the title field decorator and say “validate” and use a predefined “required” validator.

```ts
@Fields.string({
    validate: Validators.required
})
title = '';
```
---

> Let's add try, catch and alert
```ts
const saveTask = async () => {
  try {
    replaceTask(await taskRepo.save(task));
  } catch (err: any) {
    alert(err.message);
  }
}
```
--- 
- Demo
>If I clear the network tab and try again, 
you can see there’s no request being sent to the backend because the validation check is done by javascript running in the browser. 

---
- Demo post call
> Now let’s try to bypass that by sending a POST request to the “tasks” API route with an empty title
> Doesn’t work and I get the same validation error from the API.

---
- Refactor to Arrow Method
> Of course, you don’t have to use a predefined validator, you can use any arrow function to validate the data, 
so I can say for example 
> if task.title.length 
is less than 3 
throw “too short””. 

---

# Let's add a `setAllCompleted` functionality

```ts
const setAllCompleted = async (completed: boolean) => {
  for (const task of await taskRepo.find()) {
    await taskRepo.save({ ...task, completed });
  }
  setTasks(await taskRepo.find());
};
```
```html
  </ul>
  <button onClick={setAllCompleted(true)}>Set All Completed</button>
  <button onClick={setAllCompleted(false)}>Set All UnCompleted</button>
</main>
```
---

# Backend method, refactor to backend
>  looking at the “setAllCompleted” function in the App component. It works but it’s not efficient, because it’s sending multiple requests to update all of these tasks.
 ---
 > One of the nice things about Remult is that we can use the same CRUD syntax both in Frontend code, to interact with the API, and in Backend code, to interact directly with the database.
--- 
>  Let’s refactor this code to run on the backend instead.

--- 

1. Add class `TasksController`
2. Copy setAllCompleted to class
3. Adjust syntax
> The BackendMethod decorator tells Remult that if I call this method from Frontend code, don’t run this code, but instead send an API request and run it on the backend.

4. Register on API
---
6. Call from front end
> And now, in the App component, instead of this loop I can simply say: await TasksController.setAll(completed). That’s it.
7. Show network tab
--- 
> One more important thing we get from this is type-safety for this API call, so if I try to send a string instead of a boolean here, TypeScript tells me I’ve made a mistake.


---
# Authentication 
>Let's see how we can use Remult to control access to the REST API using simple, declarative code.

--- 
>Let’s say I want to restrict the API, so that only authenticated users can create, read and update tasks



```ts
@Entity("tasks", {
    allowApiCrud: Allow.authenticated
})
```

--- 
- Show request fail
> As soon as I save this my page will fail because I’m not authenticated. Let’s add a “sign-in” feature so we can make this app usable again.

---
> you can choose any way you like to implement an authentication flow. All you have to tell Remult is how to extract the current user from any incoming request.
  
> For this demo I’m going to use the Express “cookie-session” middleware,

--- 
Add session
```sh
npm i cookie-session
npm i --save-dev @types/cookie-session
```
```ts
import session from 'cookie-session';
//...
app.use(session({
    secret: "my secret"
}));
```
---

# create `server/auth.ts`
```ts
export const auth = Router();
auth.use(json());
const validUsers:UserInfo[] = [
    { id: "1", name: "Jane" },
    { id: "2", name: "Steve" },
];
```
```ts
auth.post('/api/signIn', (req, res) => {
  const user = validUsers.find(user => user.name === req.body.username);
  if (user) {
    req.session!['user'] = user;
    res.json(user);
  } else {
    res.status(404).json("Invalid user, try 'Steve' or 'Jane'")
  }
});
```
--- 
register the `auth` on the server
> Real-life authentication flows can be  more complex than this, but for a demo this will do.

```ts
app.use(auth);
```

---

# Add `Auth.tsx` to `src`
```tsx
const Auth: React.FC<PropsWithChildren> = ({ children }) => {
  return <>{children}</>
}

export default Auth;
```
- In main.tsx, wrap with Auth
```ts
<React.StrictMode>
  <Auth>
    <App />
  </Auth>
</React.StrictMode>
```

---
# Back to Auth.tsx
>Now let’s go back to the frontend and add user sign-in.

```ts
const [currentUser, setCurrentUser] = useState<UserInfo>();
if (!currentUser)
  return <>
    <h1>Sign In</h1>
  </>
return <>{children}</>
```
--- 
```tsx
const [username, setUsername] = useState("");
  //...
<main>
  <input value={username}
    onChange={e => setUsername(e.target.value)}
    placeholder="Type Steve or Jane"
  />
  <button>Sign In</button>
</main>
```

---
# Sign in
```ts
const signIn = async () => {
  const result = await fetch('/api/signIn', {
    method: "POST",
    headers: {
      'Content-type': 'application/json'
    },
    body: JSON.stringify({ username })
  })
  if (result.ok) {
    setCurrentUser(await result.json())
    setUsername('');
  }
  else
    alert(await result.json());
}
```
```
<button onClick={signIn}>Sign In</button>
```

---
# Demo
1. Still no data
> I need to tell Remult how to extract the current user from a request. For that, I’ll set the getUser option here to a function that returns the “request.session.user”.

getUser:request=>request.session!['user'],

---

# On refresh, it requires signing in again,
## server `auth.ts`
```ts
auth.get("/api/currentUser", (req, res) =>
    res.json(req.session!['user'])
);
```
## Client `Auth.tsx`
```ts
useEffect(() => {
  fetch('/api/currentUser').then(async r => {
    setCurrentUser(await r.json());
  })
}, []);
```

---
# Sign out
server `auth.ts`
```ts
auth.post("/api/signOut", (req, res) => {
  req.session!['user'] = null;
  res.sendStatus(200);
});
```
Client `Auth.tsx`
```tsx
const signOut = async () => {
  await fetch('/api/signOut', {method: "POST"});
  setCurrentUser(undefined);
}
```
- continue on the next page

---

```tsx
return <>
  <header style={{
    display: 'flex',
    justifyContent: 'space-between',
    alignItems: 'center'
  }}>
    Hello {currentUser.name} <button>Sign Out</button>
  </header>
  {children}
</>
```

---

# Role-based Authorization
Let’s go back to the Task entity and add another restriction -  only users who have an “admin” role can delete tasks.

```ts
@Entity("tasks", {
  //...
  allowApiDelete:"admin"
})
```
`auth.ts`
```ts
{ id: "1", name: "Jane", roles: ["admin"] },
```
---
# Database

```sh
npm i pg
```

```ts
//...
import { createPostgresConnection } from "remult/postgres";
export const api = remultExpress({
     //...
     dataProvider: createPostgresConnection({
         connectionString: "your connection string"
     }),
});
```
---
# Deployment - adjust the build
```json
{
  "compilerOptions": {
    //...
    "noEmit": false,
    "outDir": "dist",
    "rootDir": "src"
  },
  "include": [
      "src/server/index.ts"
  ]
}
```
--- 
# Adjust package json
```
"build": "... && tsc -p tsconfig.server.json",
"start":"node dist/server/"
```
---

# Use express to return the react files
We want to use the same nodejs server to return the static files etc...
```ts
//...
const frontendFiles = process.cwd()+'/dist';
app.use(express.static(frontendFiles));
app.get('/*', (_, res) => {
  res.send(frontendFiles + '/index.html');
});

```

--- 
# Build and test
1. Stop the server terminal
2. npm run build
3. npm run start
   

---
# Replace hard coded strings with environment variables
fix session secret 
```ts
secret: process.env['SESSION_SECRET'] || 'my secret'
```
```ts
app.listen(process.env["PORT"] || 3002
```
```ts
connectionString: process.env['DATABASE_URL'] || "...."
```
---

# Deploy to railway.app
1. Push to github as public repo
2. Open `railway.app`
3. Start new project
4. Provision databases
5. New, 
   1. github repo
6. Click on project,
   1. variables
      1. SESSION_SECRET - online uuid generator
   2. Settings
      1. Generate domain