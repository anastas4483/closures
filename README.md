## Определение
_Замыкание_ - особенность JavaScript, позволяющая использовать Scope функции в момент ее создания при вызове.  
Базовый пример:
```js
function getNumberInDegree(degree) {
  return function() {
    console.log(num*degree);
  }
}

const sqrt = getNumberInDegree(2);
const result = sqrt(3) // 9
```
Таким образом возвращаемая функция "запомнила" свое окружение при инициализации (на строчке `const sqrt = getNumberInDegree(2)`). После этого при ее вызове внутри всегда degree равен 2

## Пример замыкания №1
Самый общий частый пример замыкания, когда в родительском компоненте  создается функция, использующая стейт этого компонента. И она передается в дочерний. Таким образом, вызывается она в другом компоненте, но значения использует те, которые были в момент ее создания.

## Пример замыкания №2
При использовании хуков также присутствует замыкание. Например:
```tsx
const useGetPageableData = () => {
  const defaultPageSize = 25;
  const [pageNumber, setPageNumber]=useState(0);

  const getPageData () => {
    dispatch(getData(defaultPageSize, pageNumber));
    // ...
  }

  return {
  getPageData
  }
}
```

При вызове getPageData внутри компонента, функция будет использовать контекст, который был при ее инициализации, а именно `defaultPageSize` и `pageNumber`

## Пример замыкания №3

```tsx
interface Item{
  name: string;
  job: string;
}

interface TableProps{
  data: Item[];
}

const Table = ({data}) => {
  const jobs = ['QA', 'Frontend', 'Backend'];

  const onChange = (item: Item, newJob: string) => {
    const updatedItem = {...item, job: newJob};
    dispatch(updateItem(updatedItem));
  };

  return(
    <table>
      <tr>
        <th>Имя</th>
        <th>Работа</th>
      </tr>
      {data.map((item) => (
        <tr>
          <td>{item.name}</td>
          <td>
            <Select
              value={item.job}
              options={jobs}
              onChange={(newJob) => onChange(item, newJob)}
            />
          </td>
        </tr>
      ))}
    </table>
  )
}
```

В компонент Table передаются данные о юзерах - их имена и сфера деятельности. Внутри таблицы в колонке "Работа" есть селект, с помощью которого можно менять значение job параметра у каждого юзера. При изменении значения в селекте, данные через dispatch обновляются. Внутри dispatch в экшен передается обновленный объект юзера.
Задача: написать функцию, onChange, которую надо передать в селект.
Решение:
	Самый простой вариант - написать функцию, получающую 2 параметра - исходный item и новое значение job.
```tsx
const onChange = (item: Item, newJob: string) => {
  const updatedItem = {...item, job: newJob};
  dispatch(updateItem(updatedItem))
};
```

Тогда передача этой функции в пропсе будет выглядеть так: `onChange={(newValue) => onChange(entity, newValue)}`.

Функцию можно переписать, добавив замыкание, таким образом:
```tsx
const onChange = (item: Item) => {
  return (newJob: string) => {
    const updatedItem = {...item, job: newJob};
    dispatch(updateItem(updatedItem))
  }
};
```

Следовательно, при передаче в пропс функции ее вызов будет выглядеть так:
```tsx
<Select
  ...
  onChange={onChange(item)}
/>
```

Этот вариант легче читать
