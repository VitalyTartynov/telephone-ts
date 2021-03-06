# telephone-ts [![Tweet](https://img.shields.io/twitter/url/http/shields.io.svg?style=social)](https://twitter.com/intent/tweet?text=Telephone-ts%20TypeScript%20Architecture%20&url=https://github.com/bobbylite/telephone-ts&hashtags=Inversion-of-Control,Events,TypeScript,TelephoneTS)
Telephone-ts is a "Event Emitter-less" TypeScript Event Architecture.  Without the use of the 'events' module from node, Telephone-ts is an OOP message bus that allows developers to easily register their TypeScript Event Handlers to listen to when an Event message is "Shouted" on the telephone line! 

Sure, there are tons of great event modules... I found this way useful when trying to keep my code modular with InversifyJS, which is the IOC DI registering engine behind Telephone-ts. 

## Run example
There is a test script available to run to see how this repo works. 
Run the following in your terminal.

### Clone the code 
```bash
git clone https://github.com/bobbylite/telephone-ts.git
cd telephone-ts/
```

### Install dependencies
```bash
npm install 
npm run test
```

## How to use in project

### Step 1
Create your Event Interfaces.  This will explain what you want your event to send over our line! 
```typescript
export interface IHelloEvent {
    msg: string;
}
```

### Step 2
Implement your Event so you can have everything you need.  Maybe you want to setup a service for handling an event later?  
In this example we'll just use our greeting string we all love. 
```typescript 
export class HelloEvent implements IHelloEvent {
    public msg: string = "Hello World!";
}
```

### Step 3 
Next we'll create our event handler interfaces. This will explain what is needed to handle the event!
```typescript 
export interface IHelloHandler {
    // anything you want your event handler to have.
}
```

### Step 4
This is where we will implement our event handler.  We must make sure to inherit/extend the BaseHandler class provided. 
```typescript
export class HelloHandler extends BaseHandler<IHelloEvent> implements IHelloHandler {

    public constructor() {
        super();
    }

    protected HandleMessage(message: IHelloEvent) : IHelloEvent{
        console.log(message);

        return message;
    }
}
```

### Step 5
We now have everything we need to register, and emit events... Or create our quiet listening wire and shout on that wire! 
Both Register and Call require the following: 
TelephonetsInstance.Register<EventInterface>("EventInterface", HandlerClassReference);
TelephonetsInstance.Call<EventInterface>("EventInterface", new EventClass);
```typescript
const sleep = (ms: number) => new Promise(resolve => setTimeout(resolve, ms));

async function Test() {
    var telephonets = new Telephonets();

    telephonets.Register<INotHelloEvent>("INotHelloEvent", NotHelloHandler);
    telephonets.Register<IHelloEvent>("IHelloEvent", HelloHandler);


    telephonets.Call<IHelloEvent>("IHelloEvent", new HelloEvent); // outputs -> HelloEvent { msg: 'Hello World!' }
    await sleep(1000);
    telephonets.Call<INotHelloEvent>("INotHelloEvent", new NotHelloEvent); // outputs -> NotHelloEvent { msg: 'Not Hello World!' }
}
```

## Behind the scenes
Behind the scenes we have two important files that really auto-wire up the events to the handlers.  These two files are the telephonets.ts and BaseHandler.ts.  telephonets uses InversifyJs' Container to auto-wire similar to how Autofac or other IOC libraries work.  The BaseHandler is what we need our custom handlers to inherit from to ensure the project is structured properly.  Take a look below.

#### telephonets.ts
```typescript
export class Telephonets implements ITelephonets {

    private container: Container;

    public constructor() {
        this.container = new Container();
    }

    public Register<T>(symbolString: string, Handler: any) : void {
        this.container.bind<T>(symbolString).to(Handler);
    }

    public Call<T>(symbolString: string, message: any) : void {
        this.container.get<IBaseHandler<T>>(symbolString).ReceiveMessage(message);
    }
}
```

#### BaseHandler.ts
```typescript
@injectable()
export abstract class BaseHandler<T> implements IBaseHandler<T> {
    public ReceiveMessage(injection: T) : void {
        try {
            this.HandleMessage(injection);
        } catch(err) {
            console.log(err);
        }
    }
    protected abstract HandleMessage(message: T): T;
}
```
