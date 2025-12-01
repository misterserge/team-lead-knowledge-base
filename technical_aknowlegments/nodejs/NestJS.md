Базовый уровень
1. Что такое NestJS и в чем его преимущества перед Express.js?

NestJS — это прогрессивный фреймворк для создания серверных приложений на Node.js, построенный на TypeScript и Express (или Fastify).

Преимущества перед Express.js:

Модульность – позволяет разбивать код на независимые модули.

Встроенная поддержка DI (Dependency Injection).

ООП, SOLID, архитектурные паттерны – NestJS следует паттерну MVC.

Из коробки поддерживает WebSockets, GraphQL, микросервисы и gRPC.

Простота тестирования – встроенная поддержка unit и e2e-тестирования.

2. Как в NestJS устроена архитектура модулей?

В NestJS все построено вокруг модулей. Приложение состоит из одного главного модуля (AppModule) и множества других модулей.

Пример:

import { Module } from '@nestjs/common';
import { UsersModule } from './users/users.module';

@Module({
  imports: [UsersModule], // Импортируем другие модули
})
export class AppModule {}

3. Что такое контроллеры в NestJS и как их создать?

Контроллеры (@Controller()) управляют входящими HTTP-запросами и отправляют ответы клиенту.

Пример контроллера:

import { Controller, Get } from '@nestjs/common';

@Controller('users')
export class UsersController {
  @Get()
  getAllUsers() {
    return ['user1', 'user2'];
  }
}

4. Как объявить роут в NestJS?

Роут объявляется в контроллере с помощью декораторов @Get(), @Post(), @Put(), @Delete().

@Controller('products')
export class ProductsController {
  @Get(':id')
  getProduct(@Param('id') id: string) {
    return `Product with id ${id}`;
  }
}

5. Что такое поставщики (Providers) в NestJS?

Поставщики (Providers) — это сервисы, репозитории и любые классы, которые можно внедрять через DI.

Пример сервиса:

import { Injectable } from '@nestjs/common';

@Injectable()
export class UsersService {
  getUsers() {
    return ['user1', 'user2'];
  }
}


Регистрация в модуле:

@Module({
  providers: [UsersService],
  exports: [UsersService], // Для использования в других модулях
})
export class UsersModule {}

6. Как в NestJS реализуется внедрение зависимостей (DI)?

DI в NestJS использует декоратор @Injectable(), а сервисы автоматически внедряются в конструктор.

Пример:

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  getUsers() {
    return this.usersService.getUsers();
  }
}

7. Как использовать интерфейсы DTO в NestJS?

DTO (Data Transfer Object) используется для строгой типизации входных данных.

Пример DTO:

export class CreateUserDto {
  name: string;
  email: string;
}


Использование в контроллере:

@Post()
createUser(@Body() createUserDto: CreateUserDto) {
  return this.usersService.create(createUserDto);
}

8. Как работает Pipe в NestJS и зачем он нужен?

Pipes используются для валидации и трансформации данных.

Пример ValidationPipe:

import { ValidationPipe } from '@nestjs/common';

@Post()
createUser(@Body(new ValidationPipe()) createUserDto: CreateUserDto) {
  return this.usersService.create(createUserDto);
}

9. Что такое Guards и как они используются для авторизации?

Guards (@Injectable()) используются для авторизации перед выполнением запроса.

Пример AuthGuard:

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return request.headers.authorization === 'Bearer token';
  }
}


Использование в контроллере:

@Get()
@UseGuards(AuthGuard)
getData() {
  return { message: 'Protected route' };
}

10. Как в NestJS организована обработка ошибок?

NestJS использует Exception Filters для обработки ошибок.

Пример HttpExceptionFilter:

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const response = host.switchToHttp().getResponse();
    response.status(exception.getStatus()).json({
      message: exception.message,
    });
  }
}


Использование:

@UseFilters(HttpExceptionFilter)
@Get()
throwError() {
  throw new HttpException('Forbidden', 403);
}

Средний уровень
11. В чем разница между модульным, глобальным и динамическим модулями в NestJS?

Обычный модуль — просто импортируется в imports.

Глобальный модуль (@Global()) — доступен во всем приложении.

Динамический модуль — создает модуль с настройками.

@Global()
@Module({
  providers: [LoggerService],
  exports: [LoggerService],
})
export class LoggerModule {}

12. Как работает Middleware в NestJS?

Middleware выполняется до того, как запрос попадет в контроллер.

export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request logged:', req.url);
    next();
  }
}


Применение в AppModule:

export class AppModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes('users');
  }
}

13. Как создать и использовать Interceptors?

Interceptors изменяют запрос или ответ.

@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(map(data => ({ result: data })));
  }
}


Применение:

@UseInterceptors(TransformInterceptor)
@Get()
getData() {
  return { message: 'Hello' };
}

Продвинутый уровень
21. Как работает микросервисная архитектура в NestJS?

NestJS поддерживает Kafka, RabbitMQ, Redis, gRPC.

Пример Kafka:

@Module({
  imports: [
    ClientsModule.register([
      { name: 'KAFKA_SERVICE', transport: Transport.KAFKA },
    ]),
  ],
})
export class AppModule {}

30. Как можно интегрировать Redis или Kafka в NestJS?

Redis как кеш:

import { CacheModule } from '@nestjs/common';

@Module({
  imports: [CacheModule.register({ store: 'redis', host: 'localhost', port: 6379 })],
})
export class AppModule {}
