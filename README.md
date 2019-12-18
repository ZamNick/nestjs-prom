

## Description

A prometheus module for Nest.

## Installation

```bash
$ npm install --save @digikare/nestjs-prom prom-client
```

## How to use

Import `PromModule` into the root `ApplicationModule`

```typescript
import { Module } from '@nestjs/common';
import { PromModule } from '@digikare/nestjs-prom';

@Module({
  imports: [
    PromModule.forRoot({
      defaultLabels: {
        app: 'my_app',
      }
    }),
  ]
})
export class ApplicationModule {}
```

### Setup metric

To setup a metric, the module provide multiple ways to get a metric.

#### Using PromService and Dependency Injection

By using `PromService` service with DI to get a metric.

```typescript
@Injectable()
export class MyService {

  private readonly _counter: CounterMetric;
  private readonly _gauge: GaugeMetric,
  private readonly _histogram: HistogramMetric,
  private readonly _summary: SummaryMetric,

  constructor(
    private readonly promService: PromService,
  ) {
    this._counter = this.promService.getCounter({ name: 'my_counter' });
  }

  doSomething() {
    this._counter.inc(1);
  }

  reset() {
    this._counter.reset();
  }
}
```

#### Using Param Decorator

You have the following decorators:

- ```@PromCounter()```
- ```@PromGauge()```
- ```@PromHistogram()```
- ```@PromSummary()```


```typescript

import { CounterMetric, PromCounter } from '@digikare/nest-prom';

@Controller()
export class AppController {

  @Get('/home')
  home(
    @PromCounter('app_counter_1_inc') counter1: CounterMetric,
    @PromCounter({ name: 'app_counter_2_inc', help: 'app_counter_2_help' }) counter2: CounterMetric,
  ) {
    counter1.inc(1);
    counter2.inc(2);
  }

  @Get('/home2')
  home2(
    @PromCounter({ name: 'app_counter_2_inc', help: 'app_counter_2_help' }) counter: CounterMetric,
  ) {
    // counter and counter2 on home method is the instance
    counter.inc(2);
  }
}
```

### Metric class instances

```typescript
@PromInstanceCounter()
export class MyClass {

}
```

Will generate a counter called: `app_MyClass_instances_total`

### Metric method calls

```typescript
@Injectable()
export class MyService {
  @PromMethodCounter()
  doMyStuff() {

  }
}
```

Will generate a counter called: `app_MyService_doMyStuff_calls_total`

### Metric endpoint

The default metrics endpoint is `/metrics` this can be changed with the customUrl option

```ts
@Module({
  imports: [
    PromModule.forRoot({
      defaultLabels: {
        app: 'my_app',
      },
      customUrl: 'custom/uri',
    }),
  ],
})
export class MyModule
```

Now your metrics can be found at `/custom/uri`.

> PS: If you have a global prefix, the path will be `{globalPrefix}/metrics` for
the moment.

## API

### PromModule.forRoot() options

- `withDefaultsMetrics: boolean (default true)` enable defaultMetrics provided by prom-client
- `withDefaultController: boolean (default true)` add internal controller to expose /metrics endpoints
- `useHttpCounterMiddleware: boolean (default false)` register http_requests_total counter

## Auth/security

I do not provide any auth/security for `/metrics` endpoints.
This is not the aim of this module, but depending of the auth strategy, you can
apply a middleware on `/metrics` to secure it.

## TODO

- Update readme
  - Gauge
  - Histogram
  - Summary
- Manage registries
- Tests
- Adding example on how to secure `/metrics` endpoint
  - secret
  - jwt

## License

MIT licensed
