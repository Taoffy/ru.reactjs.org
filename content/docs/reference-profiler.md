---
id: profiler
title: API для работы с Profiler
layout: docs
category: Reference
permalink: docs/profiler.html
---

`Profiler` измеряет то, как часто рендерится React-приложение и какова «стоимость» этого.
Его задача — помочь найти медленные части приложения, которые можно оптимизировать (например, через [мемоизацию](https://ru.reactjs.org/docs/hooks-faq.html#how-to-memoize-calculations)).

> Примечание:
>
> Профилирование добавляет накладные расходы, поэтому **оно отключено в [продакшен-режиме](https://ru.reactjs.org/docs/optimizing-performance.html#use-the-production-build)**.
> 
> Для отладки на продакшене, React предоставляет специальную продакшен-сборку с включенным профилированием.
> Подробнее об использовании данной сборки можно узнать на [fb.me/react-profiling](https://fb.me/react-profiling).

## Использование {#usage}

`Profiler` может быть добавлен в любую часть React-дерева для измерения стоимости рендеринга этой части.
Он принимает два пропа: `id` (string) и колбэк `onRender` (function), который React вызывает каждый раз, когда компонент внутри дерева «фиксирует» обновление. 

Например, так выглядит процесс профилирования компонента `Navigation` и его дочерних компонентов:

```js{3}
render(
  <App>
    <Profiler id="Navigation" onRender={callback}>
      <Navigation {...props} />
    </Profiler>
    <Main {...props} />
  </App>
);
```

Для замера разных частей приложения могут быть использованы несколько компонентов `Profiler`:
```js{3,6}
render(
  <App>
    <Profiler id="Navigation" onRender={callback}>
      <Navigation {...props} />
    </Profiler>
    <Profiler id="Main" onRender={callback}>
      <Main {...props} />
    </Profiler>
  </App>
);
```

Также `Profiler` может быть вложенным с целью замера разных компонентов внутри поддерева:
```js{2,6,8}
render(
  <App>
    <Profiler id="Panel" onRender={callback}>
      <Panel {...props}>
        <Profiler id="Content" onRender={callback}>
          <Content {...props} />
        </Profiler>
        <Profiler id="PreviewPane" onRender={callback}>
          <PreviewPane {...props} />
        </Profiler>
      </Panel>
    </Profiler>
  </App>
);
```

> Примечание:
>
> Несмотря на то, что компонент `Profiler` достаточно легковесный, его следует использовать только при необходимости; каждое его использование увеличивает нагрузку на CPU и память. 

## Колбэк `onRender` {#onrender-callback}

`Profiler` принимает функцию `onRender` в качестве пропа.
React вызывает эту функцию каждый раз, когда компонент внутри профилируемого дерева «фиксирует» изменение.
Эта функция принимает параметры, которые описывают, что было отрендерено и сколько времени это заняло.

```js
function onRenderCallback(
  id, // проп "id" из дерева компонента Profiler, для которого было зафиксировано изменение
  phase, // либо "mount" (если дерево было смонтировано), либо "update" (если дерево было повторно отрендерено)
  actualDuration, // время, затраченное на рендер зафиксированного обновления
  baseDuration, // предполагаемое время рендера всего поддерева без кеширования
  startTime, // когда React начал рендерить это обновление
  commitTime, // когда React зафиксировал это обновление
  interactions // Множество «взаимодействий» для данного обновления 
) {
  // Обработка или логирование результатов...
}
```

Давайте поближе рассмотрим каждый из пропсов:

* **`id: string`** - 
Проп `id` из дерева компонента `Profiler`, для которого было зафиксировано изменение.
Может использоваться для определения той части дерева, которое было зафиксировано, если вы используете несколько профилировщиков.
* **`phase: "mount" | "update"`** -
Показывает, было ли дерево только что смонтировано в первый раз или повторно отрендерено в результате изменения пропсов, состояния или хуков.
* **`actualDuration: number`** -
Время, затраченное на рендеринг компонента `Profiler` и его дочерних компонентов для текущего обновления.
Показывает насколько хорошо поддерево использует мемоизацию (например, [`React.memo`](/docs/react-api.html#reactmemo), [`useMemo`](/docs/hooks-reference.html#usememo), [`shouldComponentUpdate`](/docs/hooks-faq.html#how-do-i-implement-shouldcomponentupdate)).
В идеальном случае это значение должно существенно снизиться после монтирования, так как многим из дочерних компонентов нужно будет перерендериваться только в случае, если изменяются их специфичные пропсы.
* **`baseDuration: number`** -
Длительность самого последнего рендеринга для каждого отдельного компонента внутри дерева компонента `Profiler`.
Это значение оценивает стоимость рендера в наихудшем случае (например, изначальное монтирование или дерево без мемоизации).
* **`startTime: number`** -
Временная метка, когда React начал рендерить текущее обновление.
* **`commitTime: number`** -
Временная метка, когда React зафиксировал текущее обновление.
Это значение доступно для всех профилировщиков при фиксации, позволяя группировать их, если в этом есть необходимость.
* **`interactions: Set`** -
<<<<<<< HEAD
Множество [«взаимодействий»](http://fb.me/react-interaction-tracing), которые были зафиксированы во время подготовки изменения (например, когда `render` или `setState` были вызваны).
=======
Set of ["interactions"](https://fb.me/react-interaction-tracing) that were being traced the update was scheduled (e.g. when `render` or `setState` were called).
>>>>>>> 5b6ad388804aaa5cf5504ccd04329f52960e17ae

> Примечание:
>
> Взаимодействия могут быть использованы для установки причины обновления, хотя API для их отслеживания все еще экспериментальное.
>
<<<<<<< HEAD
> Вы можете узнать подробнее на [fb.me/react-interaction-tracing](http://fb.me/react-interaction-tracing).
=======
> Learn more about it at [fb.me/react-interaction-tracing](https://fb.me/react-interaction-tracing)
>>>>>>> 5b6ad388804aaa5cf5504ccd04329f52960e17ae
