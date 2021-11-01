# Аргументы

По умолчанию `key` будет передан в `fetcher` в качестве аргумента. Итак, следующие 3 выражения эквивалентны:

```js
useSWR('/api/user', () => fetcher('/api/user'))
useSWR('/api/user', url => fetcher(url))
useSWR('/api/user', fetcher)
```

## Множественные аргументы

В некоторых сценариях полезно передать несколько аргументов (может быть любое значение или объект) в `fetcher`. Например, запрос на выборку с авторизацией:

```js
useSWR('/api/user', url => fetchWithToken(url, token))
```

Это **неверно**. Поскольку идентификатор (также ключ кеша) данных - это `'/api/user'`,
даже если `token` изменится, SWR все равно будет использовать тот же ключ и возвращать неверные данные.

Вместо этого вы можете использовать **массив** в качестве параметра `key`, который содержит несколько аргументов `fetcher`:

```js
const { data: user } = useSWR(['/api/user', token], fetchWithToken)
```

Функция `fetchWithToken` по-прежнему принимает те же 2 аргумента, но ключ кеша теперь также будет связан с `token`.

## Передача объектов

Скажем, у вас есть другая функция, которая извлекает данные в рамках пользователя: `fetchWithUser(api, user)`. Вы можете сделать следующее:

```js
const { data: user } = useSWR(['/api/user', token], fetchWithToken)
// ...и передать его как аргумент другому запросу
const { data: orders } = useSWR(user ? ['/api/orders', user] : null, fetchWithUser)
```

Ключом запроса теперь является комбинация обоих значений. SWR **поверхностно** сравнивает аргументы на каждом рендере и запускает повторную проверку, если какой-либо из них изменился.  
Имейте в виду, что вы не должны воссоздавать объекты при рендеринге, так как они будут обрабатываться как разные объекты при каждом рендеринге:

```js
// Не делайте этого! Зависимости будут изменяться при каждом рендере.
useSWR(['/api/user', { id }], query)

// Вместо этого вы должны передавать только «стабильные» значения.
useSWR(['/api/user', id], (url, id) => query(url, { id }))
```

Дэн Абрамов очень хорошо объясняет зависимости [в этом посте](https://overreacted.io/a-complete-guide-to-useeffect/#but-i-cant-put-this-function-inside-an-effect).