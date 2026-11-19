Діаграма показує, як сторінки в `app/[lang]/` звертаються до маршрутів у `app/api/`, які маршрути резолвлять сервіси з `app/services/` через контейнер `app/di/` (`container.ts`, `modules/composedServices.module.ts`, `modules/repositories.module.ts`), і від яких репозиторіїв та моделей у `app/infrastructure/` ці сервіси залежать. Вузли підписані реальними папками й файлами. Маршрути `collaboration/route.ts` і `blob-url/route.ts` не ходять у сервіси й базу: перший лише валідує тіло запиту, другий віддає файли з `public/`.

```mermaid
flowchart TD
  subgraph pages ["app/[lang]"]
    archivePage["archive/page.tsx"]
    artistryPage["artistry/page.tsx"]
    researchPage["research/page.tsx"]
    cooperationPage["cooperation/page.tsx"]
    contactsPage["contacts/ContactsInfo/ContactsInfo.tsx"]
    layoutPage["layout.tsx"]
  end

  subgraph api ["app/api"]
    fundsRoute["funds/route.ts"]
    compData["compositions/data/route.ts"]
    compFilters["compositions/filters/route.ts"]
    compTitles["compositions/titles/route.ts"]
    sciData["scientific-works/data/route.ts"]
    sciFilters["scientific-works/filters/route.ts"]
    sciTitles["scientific-works/titles/route.ts"]
    collabRoute["collaboration/route.ts"]
    blobRoute["blob-url/route.ts"]
  end

  subgraph services ["app/services"]
    fundsSvc["funds/fundsService.ts"]
    artistrySvc["artistry/artistryService.ts"]
    sciSvc["scientific-works/scientificWorks.ts"]
  end

  subgraph repos ["app/infrastructure/repositories"]
    fundsRepo["funds/funds.repository.ts"]
    compRepo["artistry/compositions.repository.ts"]
    sciRepo["scientific-works/scientificWorks.repository.ts"]
  end

  subgraph models ["app/infrastructure/models"]
    fundModel["archive/Fund.ts"]
    caseModel["archive/Case.ts"]
    compModel["artistry/artistryTableData.ts"]
    genreModel["artistry/artistryGenreData.ts"]
    catModel["artistry/artistryCategoriesData.ts"]
    opusModel["artistry/artistryOpusData.ts"]
    worksModel["scientific-works/scientificWorksTableData.ts"]
    authorModel["scientific-works/scientificWorksAuthor.ts"]
  end

  archivePage --> fundsRoute --> fundsSvc --> fundsRepo
  fundsRepo --> fundModel
  fundsRepo --> caseModel

  artistryPage --> compData --> artistrySvc
  artistryPage --> compFilters --> artistrySvc
  artistryPage --> compTitles --> artistrySvc
  artistryPage --> blobRoute
  layoutPage --> blobRoute
  artistrySvc --> compRepo
  compRepo --> compModel
  compRepo --> genreModel
  compRepo --> catModel
  compRepo --> opusModel

  researchPage --> sciData --> sciSvc
  researchPage --> sciFilters --> sciSvc
  researchPage --> sciTitles --> sciSvc
  sciSvc --> sciRepo
  sciRepo --> worksModel
  sciRepo --> authorModel

  cooperationPage --> collabRoute
  contactsPage --> collabRoute
```
