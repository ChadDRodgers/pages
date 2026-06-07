AI Generated

Here is a complete, production-ready README.md file that you can copy, paste, and publish directly to your personal GitHub repository. It synthesizes all the steps we discussed into a single, comprehensive guide for your family and future reference.

------------------------------

# 👨‍👩‍👧‍👦 Secure Shared Family Todo PWA (AWS Amplify Gen 2 & TypeScript)

A secure, real-time, cross-platform collaborative Todo application designed specifically for families. This application runs on AWS Serverless infrastructure (qualifying for the AWS Free Tier), supports individual chore assignment, offers complete workspace cleanups, works on desktop browsers, and installs natively onto iPhones as a Progressive Web App (PWA) with live **Web Push Notifications** (iOS 16.4+).

## 🏗️ Architecture Overview- **Frontend Platform**: React (Vite) + TypeScript.

- **Hosting & Infrastructure**: AWS Amplify Gen 2 (Automated CI/CD linked via GitHub).
- **Authentication**: Amazon Cognito (Enforces isolated login pools and multi-factor authentication).
- **Database & Live Sync**: Amazon DynamoDB + AWS AppSync GraphQL (Provides instant real-time data pushes across family devices).
- **Push Engine**: Background Web Service Workers + AWS Lambda Functions communicating via the standard Web Push protocol (VAPID).

## 🛠️ Phase 1: Local Environment Initialization### 

1. Install PrerequisitesEnsure you have the following free tools configured on your local workstation:

- **Node.js (LTS)**: Download from [nodejs.org](https://nodejs.org).
- **AWS Account**: Register an administrative profile on the [AWS Management Console](https://amazon.com).
- **AWS CLI**: Setup locally following the [AWS CLI Installation Guide](https://amazon.com).

2. Scaffold the Project SpaceOpen your terminal and execute the following sequences to create a lightweight React project powered by Vite:

```bash
npm create vite@latest family-todo -- --template react-ts
cd family-todo
npm install
```

3. Initialize AWS Amplify Gen 2Initialize the modern code-first AWS Amplify environment:

```bash
npm create amplify@latest -y
```

*This configures an `amplify` structural folder inside your root working directory.*

## 🗄️ Phase 2: Configuration & Database Design

1. Declare the Shared Schema & Device TablesOpen your `amplify/data/resource.ts` file. Replace its complete contents with the code below. This setup allows any authenticated family member to see, update, or clear any items, and introduces a schema table to log device subscription details for push tracking.

```typescript
import { type ClientSchema, a, defineData } from '@aws-amplify/backend';

const schema = a.schema({
  Todo: a
    .model({
      content: a.string().required(),
      isDone: a.boolean().default(false),
      createdBy: a.string().required(),
      assignedTo: a.string().required(),
    })
    .authorization((allow) => [allow.authenticated()]),

  DeviceSubscription: a
    .model({
      username: a.string().required(),
      subscriptionJson: a.string().required(),
    })
    .authorization((allow) => [allow.authenticated()]),
});

export type Schema = ClientSchema<typeof schema>;
export const data = defineData({ schema });
```

2. Generate Cryptographic VAPID KeysTo authenticate your notification stream safely, run this utility command inside your terminal:

```bash
npx web-push generate-vapid-keys
```

*Save both generated string blocks safely. You will insert the **Public Key** into the React client code, and the **Private Key** securely into your AWS Lambda background handler.*

## 💻 Phase 3: Writing the Frontend Code

Install the official AWS interface and functional wrapper libraries first:```bash
npm install @aws-amplify/ui-react aws-amplify

1. Create the Service Worker HandlerCreate a file named **`sw.js`** inside your **`public/`** directory. This background listener processes background signals and launches the native iOS banner message components.

```javascript
// public/sw.js
self.addEventListener('push', (event) => {
  const data = event.data ? event.data.json() : { title: 'Family Todo Update', body: 'Someone updated the task list!' };
  
  const options = {
    body: data.body,
    icon: '/apple-touch-icon.png',
    badge: '/apple-touch-icon.png',
    vibrate: [100, 50, 100],
    data: { url: self.location.origin }
  };

  event.waitUntil(self.registration.showNotification(data.title, options));
});

self.addEventListener('notificationclick', (event) => {
  event.notification.close();
  event.waitUntil(
    clients.matchAll({ type: 'window' }).then((clientList) => {
      for (const client of clientList) {
        if (client.url === event.notification.data.url && 'focus' in client) return client.focus();
      }
      if (clients.openWindow) return clients.openWindow(event.notification.data.url);
    })
  );
});
```

2. Assemble the Application ShellOpen **`src/App.tsx`** and replace its code with the complete block below. Customize the `FAMILY_MEMBERS` array and add your unique public VAPID string.


```tsx
import { useEffect, useState } from 'react';
import { generateClient } from 'aws-amplify/data';
import { Authenticator } from '@aws-amplify/ui-react';
import '@aws-amplify/ui-react/styles.css';
import { Amplify } from 'aws-amplify';
import type { Schema } from '../amplify/data/resource';
import outputs from '../amplify_outputs.json';

Amplify.configure(outputs);
const client = generateClient<Schema>();

// 💡 UPDATE THIS LIST WITH YOUR FAMILY NAMES
const FAMILY_MEMBERS = ['Dad', 'Mom', 'Alex', 'Emily', 'Everyone'];

// Push Notification Custom Hook
function usePushNotifications(currentUserName: string) {
  // 💡 PASTE YOUR GENERATED PUBLIC VAPID KEY HERE
  const VAPID_PUBLIC_KEY = 'YOUR_PUBLIC_VAPID_KEY_HERE';

  useEffect(() => {
    if (!('serviceWorker' in navigator) || !('PushManager' in window)) return;

    async function registerNotificationDevice() {
      try {
        const registration = await navigator.serviceWorker.register('/sw.js');
        const permission = await Notification.requestPermission();
        if (permission !== 'granted') return;

        let subscription = await registration.pushManager.getSubscription();
        if (!subscription) {
          subscription = await registration.pushManager.subscribe({
            userVisibleOnly: true,
            applicationServerKey: VAPID_PUBLIC_KEY
          });
        }

        await client.models.DeviceSubscription.create({
          username: currentUserName,
          subscriptionJson: JSON.stringify(subscription)
        });
      } catch (error) {
        console.error('Push integration failed:', error);
      }
    }

    if (currentUserName && currentUserName !== 'Family Member') {
      registerNotificationDevice();
    }
  }, [currentUserName]);
}

export default function App() {
  return (
    <Authenticator>
      {({ signOut, user }) => {
        const currentUserName = user?.signInDetails?.loginId || user?.username || 'Family Member';
        usePushNotifications(currentUserName);
        
        const [todos, setTodos] = useState<Schema['Todo']['type'][]>([]);
        const [newTask, setNewTask] = useState('');
        const [assignee, setAssignee] = useState('Everyone');
        const [hideCompleted, setHideCompleted] = useState(false);
        const [viewFilter, setViewFilter] = useState<'all' | 'mine'>('all');

        useEffect(() => {
          const sub = client.models.Todo.observeQuery().subscribe({
            next: (data) => setTodos([...data.items]),
          });
          return () => sub.unsubscribe();
        }, []);

        async function handleAddTask(e: React.FormEvent) {
          e.preventDefault();
          if (!newTask.trim()) return;
          await client.models.Todo.create({
            content: newTask,
            isDone: false,
            createdBy: currentUserName,
            assignedTo: assignee,
          });
          setNewTask('');
          setAssignee('Everyone');
        }

        async function toggleTodoStatus(todo: Schema['Todo']['type']) {
          await client.models.Todo.update({ id: todo.id, isDone: !todo.isDone });
        }

        async function deleteTodo(id: string) {
          await client.models.Todo.delete({ id });
        }

        async function clearCompletedTasks() {
          const completedTasks = todos.filter(t => t.isDone);
          for (const todo of completedTasks) {
            await client.models.Todo.delete({ id: todo.id });
          }
        }

        const filteredTodos = todos.filter((todo) => {
          if (hideCompleted && todo.isDone) return false;
          if (viewFilter === 'mine') return todo.assignedTo === currentUserName || todo.assignedTo === 'Everyone';
          return true;
        });

        return (
          <main style={styles.container}>
            <header style={styles.header}>
              <h1>👨‍👩‍👧‍👦 Family Workspace</h1>
              <div style={styles.userInfo}>
                <span>Logged in: <strong>{currentUserName}</strong></span>
                <button onClick={signOut} style={styles.signOutBtn}>Sign Out</button>
              </div>
            </header>

            <form onSubmit={handleAddTask} style={styles.formCard}>
              <input
                type="text"
                value={newTask}
                onChange={(e) => setNewTask(e.target.value)}
                placeholder="What needs to be done?..."
                style={styles.input}
              />
              <div style={styles.formRow}>
                <label style={styles.label}>
                  Assignee:
                  <select value={assignee} onChange={(e) => setAssignee(e.target.value)} style={styles.select}>

{FAMILY_MEMBERS.map(m => {m})}


Add Task



<button onClick={() => setViewFilter('all')} style={{...styles.filterBtn, backgroundColor: viewFilter === 'all' ? '#007bff' : '#eee', color: viewFilter === 'all' ? 'white' : 'black'}}>All
<button onClick={() => setViewFilter('mine')} style={{...styles.filterBtn, backgroundColor: viewFilter === 'mine' ? '#007bff' : '#eee', color: viewFilter === 'mine' ? 'white' : 'black'}}>My Tasks

<div style={{ display: 'flex', alignItems: 'center', gap: '12px' }}>

<input type="checkbox" checked={hideCompleted} onChange={(e) => setHideCompleted(e.target.checked)} /> Hide Done

{todos.some(t => t.isDone) && (
🧹 Clear
)}


{filteredTodos.map((todo) => (

<div style={styles.todoContent} onClick={() => toggleTodoStatus(todo)}>
<input type="checkbox" checked={todo.isDone || false} readOnly style={styles.checkbox} />

<span style={{ textDecoration: todo.isDone ? 'line-through' : 'none', color: todo.isDone ? '#888' : '#000' }}>{todo.content}

👤 {todo.assignedTo}
By: {todo.createdBy}



<button onClick={() => deleteTodo(todo.id)} style={styles.deleteBtn}>🗑️

))}


);
}}

);
}
const styles = {
container: { maxWidth: '500px', margin: '0 auto', padding: '15px', fontFamily: 'system-ui, sans-serif' },
header: { borderBottom: '2px solid #f0f0f0', paddingBottom: '15px', marginBottom: '20px' },
userInfo: { display: 'flex', justifyContent: 'space-between', alignItems: 'center', fontSize: '14px', marginTop: '5px' },
signOutBtn: { background: '#dc3545', color: 'white', border: 'none', padding: '6px 12px', borderRadius: '6px', cursor: 'pointer' },
formCard: { background: '#f8f9fa', padding: '15px', borderRadius: '8px', border: '1px solid #e9ecef', marginBottom: '20px' },
input: { width: '100%', padding: '12px', fontSize: '16px', borderRadius: '6px', border: '1px solid #ced4da', boxSizing: 'border-box' as const, marginBottom: '10px' },
formRow: { display: 'flex', justifyContent: 'space-between', alignItems: 'center', gap: '10px' },
label: { fontSize: '14px', display: 'flex', alignItems: 'center', gap: '8px' },
select: { padding: '8px', borderRadius: '6px', border: '1px solid #ced4da', background: 'white' },
addBtn: { background: '#28a745', color: 'white', border: 'none', padding: '10px 20px', borderRadius: '6px', cursor: 'pointer', fontWeight: 'bold' },
filterBar: { display: 'flex', justifyContent: 'space-between', alignItems: 'center', marginBottom: '15px' },
buttonGroup: { display: 'flex', gap: '5px' },
filterBtn: { border: 'none', padding: '6px 12px', borderRadius: '20px', cursor: 'pointer', fontSize: '13px' },
checkboxLabel: { fontSize: '13px', display: 'flex', alignItems: 'center', gap: '5px' },
clearBtn: { background: '#6c757d', color: 'white', border: 'none', padding: '6px 10px', borderRadius: '20px', cursor: 'pointer', fontSize: '12px' },
list: { listStyle: 'none', padding: 0 },
listItem: { display: 'flex', justifyContent: 'space-between', alignItems: 'center', padding: '12px', border: '1px solid #e9ecef', background: 'white', borderRadius: '8px', marginBottom: '10px' },
todoContent: { display: 'flex', alignItems: 'flex-start', gap: '12px', flex: 1, cursor: 'pointer' },
checkbox: { width: '20px', height: '20px', cursor: 'pointer' },
textWrapper: { display: 'flex', flexDirection: 'column' as const, gap: '4px' },
meta: { display: 'flex', alignItems: 'center', gap: '10px' },
badge: { fontSize: '11px', background: '#e1ecf4', color: '#39739d', padding: '2px 6px', borderRadius: '4px' },
createdBy: { fontSize: '11px', color: '#6c757d' },
deleteBtn: { background: 'none', border: 'none', cursor: 'pointer' }
};
```

3. Add Custom Homescreen Metadata Icons

Place a square .png graphic file sized to exactly 180x180 pixels into your public/ directory folder and name it apple-touch-icon.png. Then open your root index.html file and declare the icon inside the <head> tags:

```html
<!-- Place this inside the <head> tags of your root index.html file -->

<!-- 📱 Custom app launcher icon for iPhone Home Screens -->
<link rel="apple-touch-icon" href="/apple-touch-icon.png">

<!-- 💻 Custom tab favicon for desktop web browsers -->
<link rel="icon" type="image/png" href="/apple-touch-icon.png">
```

------------------------------

## 🚀 Phase 4: CI/CD Live Deployment

1. Initialize git locally, link your new private GitHub repository, and push the source codebase:
  
```bash
git init git add . git commit -m "Initialize secure family sync app" 
git branch -M main git remote add origin YOUR_GITHUB_REPO_URL 
git push -u origin main
```

2. Navigate to the online AWS Amplify Console Workspace.
3. Choose Create New App, pick GitHub as the source repository tracking mechanism, and authorize connection permissions.
4. Select your application repository and the main branch, then click deploy. AWS will provision and deploy your database systems, Cognito user portals, and app code automatically.

------------------------------

## 🔒 Phase 5: Locking Down Server Security
To prevent random internet users from registering to your application workspace, implement these manual cloud dashboard steps:

1. Open the Amazon Cognito Console via your AWS account.
2. Choose the unique User Pool instance provisioned by Amplify.
3. Access Sign-in experience configuration settings and change Self-registration to Disabled.
4. Click Create User manually to explicitly generate distinct accounts for your individual family members.
5. Inside Security options, configure Multi-factor Authentication (MFA) to Required and pick authenticator applications as the secure identification method.

------------------------------

## 🌐 Phase 6: Custom Domain Configurations
To lock in an easy-to-remember domain like ourfamilytasks.com instead of relying on long generated hashes:

1. Inside your AWS Amplify Console workspace configuration space, click Domain management from the sidebar menu layout.
2. Select Add domain, enter your root name domain structure (e.g., ourfamilytasks.com), and click configure.
3. Point your top-level domain records and any selected www configurations down toward the primary main production branch environment.
4. Save adjustments. Amplify will handle generating a free SSL/TLS encryption certificate automatically.

------------------------------

## 🔔 Phase 7: App Stream Alerts Engine (AWS Lambda)

To route outbound mobile notifications seamlessly whenever someone assigns a task, configure an automatic event pipeline:

1. Open the AWS Lambda Console and click Create Function named SendFamilyPushNotification using Node.js runtime.
2. Inside your DynamoDB panel console, find your database table matching Todo and verify that DynamoDB Streams tracking is active (set to capture New Image properties).
3. Tie that DynamoDB Stream resource directly to your Lambda instance as its primary execution trigger.
4. Paste this code block inside the Lambda editor script to fire notification events down to active target browser clients using the internal web-push library:

```javascript

const webPush = require('web-push');
const { DynamoDBClient } = require("@aws-sdk/client-dynamodb");
const { DynamoDBDocumentClient, scan } = require("@aws-sdk/lib-dynamodb");

// 💡 Insert your VAPID keys here
const VAPID_PUBLIC_KEY = 'YOUR_PUBLIC_VAPID_KEY_HERE';
const VAPID_PRIVATE_KEY = 'YOUR_PRIVATE_VAPID_KEY_HERE';
webPush.setVapidDetails('mailto:family@example.com', VAPID_PUBLIC_KEY, VAPID_PRIVATE_KEY);
const ddbClient = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(ddbClient);

exports.handler = async (event) => {
// Scan for all logged active mobile client subscriptions inside the table
const scanParams = { TableName: process.env.SUBSCRIPTION_TABLE_NAME };
const subscriptionsData = await docClient.send(new scan(scanParams));
for (const record of event.Records) {
if (record.eventName === 'INSERT') {
const newTask = record.dynamodb.NewImage;
const content = newTask.content.S;
const assignee = newTask.assignedTo.S;
const payload = JSON.stringify({
title: '📌 New Family Task!',
body: ${assignee}: ${content}
});

for (const subItem of subscriptionsData.Items) {
const subDetail = JSON.parse(subItem.subscriptionJson);

try {
await webPush.sendNotification(subDetail, payload);
} catch (err) {
console.error("Expired device signature encountered:", err);
}
}
}
}
};
```
------------------------------

## 📱 Phase 8: Deploying to iPhones

Web Push functionality requires your app to run as an independent homescreen wrapper application:

1. Launch Safari on your target iOS iPhone device.
2. Browse out to your custom encrypted application landing URL (https://ourfamilytasks.com).
3. Tap the central Share square icon tray option located along the layout footer.
4. Scroll down and choose Add to Home Screen.
5. Launch the application from your iPhone home screen. Sign in, and grant permissions when the system prompts you to Allow Notifications.



