This library is created because currently react-native-pinch library is not maintaining actively. Also this library supports HTTP and HTTPS. 

# React Native Pinch New 👌

Callback and promise based HTTP client that supports SSL pinning for React Native.

## Installation

Using NPM:
```
npm install react-native-pinch-new
```

Using Yarn:
```
yarn add react-native-pinch-new
```
## Supports the new Autolinking

## Automatically link (Only recommended when autolinking failed)

#### With React Native 0.27+

```shell
react-native link react-native-pinch-new
```

#### With older versions of React Native

You need [`rnpm`](https://github.com/rnpm/rnpm) (`npm install -g rnpm`)

```shell
rnpm link react-native-pinch-new
```

## Manually link

### iOS (via Cocoa Pods)
Add the following line to your build targets in your `Podfile`

`pod 'react-native-pinch-new', :path => '../node_modules/react-native-pinch-new'`

Then run `pod install`

### Android

- in `android/app/build.gradle`:

```diff
dependencies {
    ...
    compile "com.facebook.react:react-native:+"  // From node_modules
+   compile project(':react-native-pinch')
}
```

- in `android/settings.gradle`:

```diff
...
include ':app'
+ include ':react-native-pinch'
+ project(':react-native-pinch').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-pinch-new/android')
```

#### With React Native 0.29+

- in `MainApplication.java`:

```diff
+ import com.localz.PinchPackage;

  public class MainApplication extends Application implements ReactApplication {
    //......

    @Override
    protected List<ReactPackage> getPackages() {
      return Arrays.<ReactPackage>asList(
+         new PinchPackage(),
          new MainReactPackage()
      );
    }

    ......
  }
```

#### With older versions of React Native:

- in `MainActivity.java`:

```diff
+ import com.localz.PinchPackage;

  public class MainActivity extends ReactActivity {
    ......

    @Override
    protected List<ReactPackage> getPackages() {
      return Arrays.<ReactPackage>asList(
+       new PinchPackage(),
        new MainReactPackage()
      );
    }
  }
```

## Adding certificates

Before you can make requests using SSL pinning, you first need to add your `.cer` files to your project's assets.

### Android

 - Place your `.cer` files under `src/main/assets/`.

### iOS

 - Place your `.der` files in your iOS Project. Don't forget to add them in your `Build Phases > Copy Bundle Resources`, in Xcode.


## Example
*Examples are using the ES6 standard*

Requests can be made by using the `fetch(url[, config, [callback]])` method of Pinch.

### Using Promises
```javascript
import pinch from 'react-native-pinch-new';

pinch.fetch('https://my-api.com/v1/endpoint', {
  method: 'post',
  headers: { customHeader: 'customValue' },
  body: '{"firstName": "Jake", "lastName": "Moxey"}',
  timeoutInterval: 10000 // timeout after 10 seconds
  sslPinning: {
    cert: 'cert-file-name', // cert file name without the `.cer`
    certs: ['cert-file-name-1', 'cert-file-name-2'], // optionally specify multiple certificates
  }
})
  .then(res => console.log(`We got your response! Response - ${res}`))
  .catch(err => console.log(`Whoopsy doodle! Error - ${err}`))
```

### Using Callbacks
```javascript
import pinch from 'react-native-pinch-new';

pinch.fetch('https://my-api.com/v1/endpoint', {
  method: 'post',
  headers: { customHeader: 'customValue' },
  body: '{"firstName": "Jake", "lastName": "Moxey"}',
  timeoutInterval: 10000 // timeout after 10 seconds
  sslPinning: {
    cert: 'cert-file-name', // cert file name without the `.cer`
    certs: ['cert-file-name-1', 'cert-file-name-2'], // optionally specify multiple certificates
  }
}, (err, res) => {
  if (err) {
    console.error(`Whoopsy doodle! Error - ${err}`);
    return null;
  }
  console.log(`We got your response! Response - ${res}`);
})
```

### Skipping validation

In original library, android did not support http. In order to achieve this, you only need to pass a valid http url.

```javascript
import pinch from 'react-native-pinch-new';

pinch.fetch('https://my-api.com/v1/endpoint', {
  method: 'post',
  headers: { customHeader: 'customValue' },
  body: '{"firstName": "Jake", "lastName": "Moxey"}',
  timeoutInterval: 10000 // timeout after 10 seconds
  ignore_ssl:true, // This can be used to ignore SSL pinning
  sslPinning: {} // omit the `cert` or `certs` key, `sslPinning` can be ommited as well
})
```

## Response Schema
```javascript
{
  bodyString: '',

  headers: {},

  status: 200,

  statusText: 'OK'
}
```

## Testing

### With jest

Using [fetch-mock](http://www.wheresrhys.co.uk/fetch-mock/) here, but nock or any other fetch polyfill would work.

```js
# __mocks__/react-native-pinch-new.js
import fetchMock from 'fetch-mock'; 

export default {
  fetch: fetchMock.sandbox(), // mock pinch's fetch with the sandbox version
};
```

```js
# __tests__/store.js
import configureMockStore from 'redux-mock-store';
import thunk from 'redux-thunk';
import pinch from 'react-native-pinch-new'; // actually the sandbox from fetch-mock

import { fetchFoos } from './path/to/store/actions';

jest.mock('react-native-pinch-new');

const middlewares = [thunk];
const mockStore = configureMockStore(middlewares);

afterEach(() => {
  pinch.fetch.reset();
  pinch.fetch.restore();
});

describe('fetchFoos', () => {
  it('creates FOO_BAR when fetching foos is done', () => {
    pinch.fetch.get(/^\/foos/, { foos: [] });
    const store = mockStore(defaultState);

    return store.dispatch(fetchFoos()).then(() => {
      expect(store.getActions()).toEqual(expect.arrayContaining(
        [expect.objectContaining({ type: FOO_BAR })],
      ));
    });
  });
});
```
