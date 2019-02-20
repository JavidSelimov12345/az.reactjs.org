---
id: context
title: Kontekst
permalink: docs/context.html
---

Kontext məlumatları komponent ağacının hər səviyyəsində proplar göndərmədən məlumatları komponentlərə ötürməyə təmin edir.

Standart React applikasiyasında, məlumat yuxarıdan aşağı (valideyndən uşağa) proplar vasitəsi ilə göndərilir. Lakin bir çox komponentin istifadə etdiyi bəzi prop tiplərinin (məsələn dil seçimi, UI şablonun) bu prop tiplərinin göndərilməsi çox yorucu və çətin ola bilər. Kontext, dəyərlərin bir komponentlər arasında, komponent ağaçının hər səviyyəsində prop göndərmədən paylaşmasını təminat edir.

- [Nə Zaman Kontekst İşlətmək Lazımdır](#when-to-use-context)
- [Kontekst İşlətməkdən Əvvəl](#before-you-use-context)
- [API](#api)
  - [React.createContext](#reactcreatecontext)
  - [Context.Provider](#contextprovider)
  - [Class.contextType](#classcontexttype)
  - [Context.Consumer](#contextconsumer)
- [Misallar](#examples)
  - [Dinamik Kontekst](#dynamic-context)
  - [Konteksti Bir Birindən Keçən Komponentdən Yeniləmək](#updating-context-from-a-nested-component)
  - [Bir Neçə Kontekstin İstehlakı](#consuming-multiple-contexts)
- [Xəbərdarlıqlar](#caveats)
- [Köhnə API](#legacy-api)

## Nə Zaman Kontekst İşlətmək Lazımdır {#when-to-use-context}

Kontekst "qlobal" qəbul olunan məlumatları (məsələn avtorizasiya olunmush istifadəçi, şablon, və ya seçilmiş dil) React komponentlər ağacında paylaşmaq üçün nəzərdə tutulmuşdur. Məsələn, aşağıdakı kodda biz Button komponentini dəyişmək üçün "theme" propunu bütün komponentlərdən bir bir göndəririk:

`embed:context/motivation-problem.js`

Kontekst işlətdikdə isə, biz propları ara elementlərdən göndərməyə bilərik:

`embed:context/motivation-solution.js`

## Kontekst İşlətməkdən Əvvəl {#before-you-use-context}

Kontekst əsasən məlumatın fərqli səviyyələrdə yerləşən *bir neçə* komponent tərəfindən işlənməsi üçündür. Bunu hər yerdə işlətməyin çünki komponentin yenidən istifadəsini çətinləşdirir.

**Əgər siz propları bir neçə səviyyədə göndərmək istəmirsinizsə, [komponent kompozisiyası](/docs/composition-vs-inheritance.html) kontekstdən daha sadə həlldir.**

Misal üçün, gəlin `user` və `avatarSize` proplarını mövcud olan `Page` komponentindan bir neçə səviyyədə göndərərkki, dərində olan `Link` və `Avatar` komponentləri bu propları oxuya bilsinlər:

```js
<Page user={user} avatarSize={avatarSize} />
// ... render edirs ...
<PageLayout user={user} avatarSize={avatarSize} />
// ... render edir ...
<NavigationBar user={user} avatarSize={avatarSize} />
// ... render edir ...
<Link href={user.permalink}>
  <Avatar user={user} size={avatarSize} />
</Link>
```

Əgər `user` və `avatarSize` propları yalnız `Avatar` komponenti tərəfindən işlənəcəksə, bu propların bütün səviyyələrdə göndərilməsi lazımsızdır. Əgər `Avatar` komponentinə yeni proplar lazım olsa, siz yenə lazım olan propları bütün ara komponentlərdən keçirməlisiniz.

Bu problemi **kontekstsiz** həll etməyin yolu [`Avatar` komponentinin özünü göndərməkdir](/docs/composition-vs-inheritance.html#containment). Bu zaman, ara komponentlərin `user` və `avatarSize` proplarını bilməsi lazım deyil:

```js
function Page(props) {
  const user = props.user;
  const userLink = (
    <Link href={user.permalink}>
      <Avatar user={user} size={props.avatarSize} />
    </Link>
  );
  return <PageLayout userLink={userLink} />;
}

// Biz bizdə aşağıdakı var
<Page user={user} />
// ... render edir ...
<PageLayout userLink={...} />
// ... render edir ...
<NavigationBar userLink={...} />
// ... render edir ...
{props.userLink}
```

Bu dəyişiklik ilə, yalnız ən yuxarıda olan Page komponenti, `Link` və `Avatar` komponentlərinin `user` və `avatarSize` proplarını işlətməyini bilir.

Bu *kontrolun inversiyası* applikasiyada göndərilən propların sayını azaldaraq və ana komponentlərə daha çox kontrol verərək, bir neçə ssenaridə kodunuzu daha təmiz edir. Lakin, bu metod hər ssenari üçün düzgün seçim deyil: mürəkkəbiliyi komponent ağacında yüksək olan komponentlərə göndərdikdə, bu yuxarı səviyyəli komponentləri daha çətinləşdirir və aşağı səviyyəli komponentlərin daha əyilgən olmasını məcbur edir.

Siz komponentdə bir uşaq göstərməyə məhdud deyilsiniz. Siz bir neçə uşağı, hətta uşaqlar üçün bir neçə "yuvada" edə bilərsiniz ([burada sənələşmişdir](/docs/composition-vs-inheritance.html#containment)):

```js
function Page(props) {
  const user = props.user;
  const content = <Feed user={user} />;
  const topBar = (
    <NavigationBar>
      <Link href={user.permalink}>
        <Avatar user={user} size={props.avatarSize} />
      </Link>
    </NavigationBar>
  );
  return (
    <PageLayout
      topBar={topBar}
      content={content}
    />
  );
}
```

Bu metod, uşaqları yaxın valideyndən ayırmaq üçün kifayətdir. Əgər uşaq komponent, render etməzdən öncə valideyn ilə kommunikasiya qurmalıdırsa, siz [render proplardan](/docs/render-props.html) istifadə edə bilərsiniz.

Lakin, bəzən eyni məlumatlar komponent ağacında fərqli səviyyələrdə bir neçə komponent tərəfindən işlədilə bilməlidirlər. Kontekst belə məlumatları və bu məlumatlarda olan dəyişiklikləri, bütün aşağısında olan komponentlərə "yayımlaya" bilir. Bir çox nümunələrdə kontekst işlətmək alternativlərdən daha sadə ola bilər (dilin seçimi, şablon, məlumat keşi).


## API {#api}

### `React.createContext` {#reactcreatecontext}

```js
const MyContext = React.createContext(defaultValue);
```

Kontekst obyekini yaradır. React Context obyektinə "abunə olan" komponentləri render edərkən, yuxarı səviyyədə ən yaxın olan `Provider`-dən cari kontext dəyərini oxuyacaq.

`defaultValue` arqumenti **yalnız** komponentin yuxarı səviyyəsində Provider olmadığı zaman işlənir. Bu komponentləri Provider ilə əhatə etmədən, ayrılıqda test etmək üçün faydalıdır. Qeyd: Provider dəyərinə `undefined` göndərildikdə işlədən komponentlər `defaultValue`-dan istifadə etmirlər.

### `Context.Provider` {#contextprovider}

```js
<MyContext.Provider value={/* bir dəyər */}>
```

Hər bir Context obyekti Provider adlı React komponenti ilə gəlir. Bu komponent kontekstdə olan dəyişikliklərə abunə olmaq istəyən komponentlərə imkan yaradır.

Provayder `value` propu qəbul edir. Bu propun dəyəri abunə olan komponentlərə ötürülür. Bir Provider bir neçə Consumer komponentə goşula bilər. Provayderlər eyni komponent ağacında bir neçə səviyyədə ola bilər. Ağacda dərində yerləşən provayderlər, yuxarıda olan provayderlərin dəyərlərini əvəz edir.

Provider-in aşağısında olan bütün istehlakçılar, Provider-in `value` propu dəyişdikdə yenidən render edir. Providerdən aşağıya məlumatların yayınlaması, `shouldComponentUpdate` funskiyasından asılı deyil. Bu deməkdirki, yuxarı komponentdə hansısa komponent yenilənməsə belə, Provider-ə abunə olan komponent yenilənəcək.

Dəyişikliklər yeni və köhnə dəyərlərin [`Object.is`](//developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description) alqoritminə bənzər bir alqoritm ilə müqayisəsi ilə təyin olunur.

> Qeyd
> 
> Dəyişikliklər `value`-a obyeki göndərdikdə problem yarada bilərlər: [Dəyişikliklər](#caveats) bölməsinə baxın.

### `Class.contextType` {#classcontexttype}

```js
class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context;
    /* mount etdikdə MyContext-in dəyəri ilə kənar effekt edin */
  }
  componentDidUpdate() {
    let value = this.context;
    /* ... */
  }
  componentWillUnmount() {
    let value = this.context;
    /* ... */
  }
  render() {
    let value = this.context;
    /* MyContext-in dəyəri ilə render edin */
  }
}
MyClass.contextType = MyContext;
```

`contextType` klass parametrinə [`React.createContext()`](#reactcreatecontext) ilə yaranmış Context obyekti təyin edilə bilər. Bu sizə ən yaxında olan Context-in dəyərini `this.context`-dən oxumağa icazə verir. Siz bu dəyişəndə olan dəyəri render funskiyası daxil olmaqla bütün render funskiyalarından istifadə edə bilərsiniz.

> Qeyd:
>
> Siz bu API ilə yalnız bir kontekstə abunə ola bilərsiniz. Əgər sizə birdən çox kontekst lazımdırsa [Bir Neçə Kontekstin İstehlakı](#consuming-multiple-contexts) bölməsindən oxuya bilərsiniz.
>
> Əgər siz eksperimental olan [public klass sahəsi sintaksisindən](https://babeljs.io/docs/plugins/transform-class-properties/) istifadə edirsinizsə, siz **static** klass sahəsindən istifadə edib `contextType`-ı inisializasiya edə bilərsiniz.


```js
class MyClass extends React.Component {
  static contextType = MyContext;
  render() {
    let value = this.context;
    /* dəyər əsasında render et */
  }
}
```

### `Context.Consumer` {#contextconsumer}

```js
<MyContext.Consumer>
  {value => /* kontekst dəyəri əsasında render et */}
</MyContext.Consumer>
```

Kontekst dəyişikliklərinə abunə olan React komponenti. Bu sizə [funskional komponent]((/docs/components-and-props.html#function-and-class-components)) ilə kontekstə abunə olmağa icazə verir.

[funskiyanın uşaq kimi olmasını](/docs/render-props.html#using-props-other-than-render) tələb edir. Bu funskiya kontekstin cari dəyərini qəbul edir və React nodu qaytarır. Funskiyaya göndərilən `value` arqumenti, komponent ağacında bu komponentə ən yaxın olan Provider-in `value` propu ilə eynidir. Əgər Provider yoxdursa, `value` arqumenti `createContext()`-ə keçirilən `defaultValue` propuna bərabər olacaq.

> Qeyd
> 
> 'funskiyanın uşaq kimi olması' patterni haqqında daha ətraflı məlumat üçün, [render proplar](/docs/render-props.html) sənədəinə baxın.

## Misallar {#examples}

### Dinamik Kontekst {#dynamic-context}

Şablon üçün dinamik dəyərlər işlədən daha kompleks misal:

**theme-context.js**
`embed:context/theme-detailed-theme-context.js`

**themed-button.js**
`embed:context/theme-detailed-themed-button.js`

**app.js**
`embed:context/theme-detailed-app.js`

### Konteksti Bir Birindən Keçən Komponentdən Yeniləmək {#updating-context-from-a-nested-component}

Bir çox vaxtda konteksti komponent ağacının dərinliklərində olan bir komponentdən yeniləmək lazım olur. Bu halda, siz kontekstdən funksiyanı consumer-lərin yeniləməsi üçün funksiya göndərə bilərsiniz:

**theme-context.js**
`embed:context/updating-nested-context-context.js`

**theme-toggler-button.js**
`embed:context/updating-nested-context-theme-toggler-button.js`

**app.js**
`embed:context/updating-nested-context-app.js`

### Bir Neçə Kontekstin İstehlakı {#consuming-multiple-contexts}

Kontekstin yenidən render edilməsinin tez olması üçün, React hər kontekst consumerinin komponent ağacında yeni bir nod yaratmalıdır. 

`embed:context/multiple-contexts.js`

Əgər 2 və ya daha çox kontekst dəyəri tez tez birlikdə işlədilirsə, siz bu kontekstlərin dəyərlərini birlikdə təmin edən xüsusi bir render prop komponenti düzəldə bilərsiniz.

If two or more context values are often used together, you might want to consider creating your own render prop component that provides both.

## Xəbərdarlıq {#caveats}

Kontesktin yenidən render etmək üçün dəyərin referensi ilə edir deyə, provider yenidən render etdikdə, consumer-lərdə istənilməyən renderlər oluna bilər. Məsələn, aşağıdakı kodda `value` üçün hər zaman yeni obyekt yarandığından, bütün consumer-lər yenidən render edirlər:

`embed:context/reference-caveats-problem.js`

Bu problemi həll etmək üçün, dəyəri valideynin state-inə qaldırın:

`embed:context/reference-caveats-solution.js`

## Köhnə API {#legacy-api}

> Qeyd
> 
> Əvvəl React eksperimental bir kontext API ilə gəlirdi. Bu köhnə API, React-in bütün 16.x buraxılışlarında dəstəklənəcək. Lakin applikasiyaların yeni API-a miqrasiya edilməyi tövsiyyə edilir. Köhnə API gələcəkdə buraxılan ƏSAS versiyalardan silinəcək. [köhnə kontekst haqqında buradan](/docs/legacy-context.html) oxuya bilərsiniz.
 
