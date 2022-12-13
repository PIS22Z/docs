Mateusz Maj 310826, PIS

# Redux

Redux to popularna biblioteka JavaScript do zarządzania stanem aplikacji. Została stworzona przez Dana Abramova i Andrew Clarka i jest często używana z Reactem do tworzenia interfejsów użytkownika.

Jedną z kluczowych zasad Redux jest to, że stan aplikacji jest przechowywany w pojedynczym, niezmiennym obiekcie zwanym magazynem (store). Oznacza to, że gdy coś w aplikacji wymaga zmiany, tworzony jest nowy obiekt stanu, zamiast modyfikować istniejący stan. Dzięki temu kod jest przewidywalny i łatwy do zrozumienia.

Redux używa tak zwanych "reducerow" do zarządzania stanem aplikacji. Reducery to czyste funkcje, które przyjmują bieżący stan i akcję i zwracają nowy stan. Oznacza to, że są one zawsze deterministyczne, co ułatwia ich testowanie i debugowanie.

Innym ważnym pojęciem w Redux jest pojęcie akcji. Akcje to zwykłe obiekty JavaScript, które opisują zdarzenie, które wystąpiło w aplikacji. Na przykład akcja może zostać wyzwolona, ​​gdy użytkownik kliknie przycisk lub gdy dane zostaną załadowane z interfejsu API. Akcje wysyłane są do store'a, gdzie są przetwarzane przez reducery w celu aktualizacji stanu.

Jedną z zalet korzystania z Redux jest to, że ułatwia śledzenie zmian w stanie aplikacji. Ponieważ wszystkie zmiany stanu są zarządzane poprzez akcje i reducery, możliwe jest śledzenie przepływu danych przez aplikację. Może to być szczególnie przydatne do debugowania i śledzenia błędów.

Redux zapewnia również warstwę oprogramowania pośredniego o nazwie „thunk", która ułatwia wykonywanie działań asynchronicznych, takich jak wykonywanie wywołań API lub interakcja z bazą danych. Thunks to funkcje, które można wysłać do sklepu, a po ich zakończeniu mogą wysyłać inne akcje. Pomaga to utrzymać kod w czystości i porządku.

Ogólnie rzecz biorąc, Redux jest potężnym narzędziem do zarządzania stanem aplikacji. Ułatwia pisanie przewidywalnego i łatwego w utrzymaniu kodu, a jego ścisłe zasady pomagają zapewnić, że kod jest zawsze poprawny. Warto również dodać, że Redux jest kompatybilny z TypeScriptem.

## Redux flow:

![](https://lh6.googleusercontent.com/Tcbm434Ayl4ToqB_Vek6BrSnsEeuD7iR0xuFlRb9inlpmtxRiN-Xuqg6RF0BlaUvq9qeyDH6BIhC7a_YFbGcZznNjAsNE5oL-3NPq57L6Cy1TmpF_haRUC77LMdH7YJkHu4sq7Cb_xbUyNDCU9faJicvXJNnn9tPiJB24J3gILn4-Gv6t0BOgXc-UOcvrQ)

## Przykład użycia:

W tym przykładzie tworzymy store i definiujemy reducery do aktualizacji stanu. Następnie używamy metody dispatch aby wysłać daną akcję do store'a, aby zaktualizować jego stan. Możemy również użyć metody getState, aby uzyskać dostęp do bieżącego stanu aplikacji w dowolnym momencie.

![](https://lh4.googleusercontent.com/StHPIuuD8a_umLfP1VGJrY60I4P_Wz7c7DHWjh6niT5Gg0oglXts3OpzSemI2Cd1oFJPBA4dTmI5eVd95w4azMcisplThMO4LLTdh77rxv7DE8jj-IS5ryPtwFr4pJk6LjVDD6-M6b66Hm_m7Owh1I80Mf3_7_tSnKwseQ2vE-LazVcPAll9L0Am51X0xw)

## Wady i zalety korzystania z Reduxa:

+) Scentralizowane zarządzanie stanem: Redux zapewnia prosty, scentralizowany magazyn do zarządzania stanem aplikacji. Ułatwia to śledzenie stanu aplikacji, zwłaszcza w przypadku dużych lub złożonych aplikacji.

+) Przewidywalne aktualizacje stanu: Redux używa ścisłego zestawu reguł do aktualizowania stanu, co ułatwia przewidywanie zmian stanu aplikacji w czasie. Może to ułatwić debugowanie aplikacji i pisanie testowalnego kodu.

+) Wsparcie społeczności: Redux ma dużą i aktywną społeczność.

-) Kod standardowy: Redux wymaga skonfigurowania dużej ilości kodu podstawowego, szczególnie w przypadku prostych aplikacji. Może to dodać niepotrzebnej złożoności do bazy kodu. (duży boilerplate)

-) Akcje Redux mogą być rozwlekłe, co może sprawić, że Twój kod będzie mniej czytelny i trudniejszy w utrzymaniu.

-) Akcje asynchroniczne: Obsługa akcji asynchronicznych w Redux może być złożona i wymaga dodatkowej konfiguracji, takiej jak użycie biblioteki oprogramowania pośredniego, takiej jak redux-thunk lub redux-saga.
