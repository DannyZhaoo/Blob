# React注意点

### 因为react是异步更新的，不应该依靠他们的value去计算下一个state

```javascript
//correct
this.setState((state, props) => {
    counter；state.counter + props.increment
})
```

## Code-Splitting

1. React.lazy

   这个只支持在browser端，不支持在ssr。只支持export default

   ```javascript
   //before
   import OtherComponent from './OtherComponent';
   //after
   const OtherComponent = React.lazy(() => import('./OtherComponent'));
   // 通过返回一个Promise来reslove modules
   ```

2. Suspense

   ```javascript
   const OtherComponent = React.lazy(() => import('./OtherComponent'));
   const AnotherComponent = React.lazy(() => import('./AnotherComponent'));
   
   function MyComponent() {
     return (
       <div>
         <Suspense fallback={<div>Loading...</div>}>
           <section>
             <OtherComponent />
             <AnotherComponent />
           </section>
         </Suspense>
       </div>
     );
   }
   ```

## 