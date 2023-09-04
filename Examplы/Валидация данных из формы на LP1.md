###### tags: #examples 
###### links: [[Examples]]
___


```js
$(document).on("click", ".orderform_form_button", function (e) {

  //Запускаем анимацию загрузки

  $("#mess_loading").show();

  //Очищаем window переменную с текстом ошибок, ее же будем использовать для проверки валидации, если она не пуста значит ничего не отправляем

  //Сами эти window массивы задаются прям сверху

  window.ERROR.length = 0;

  window.ERRORTEXT.length = 0;

  //Заранее создаем пустые переменные чтобы с ними работать в последствии

  let text = '';

  let price_from = '';  

  let price_to = '';

  let price_difference = '';  

  //Вносим в переменную все input которые имеются в родительском классе, и через запятую исключаем те что нам не нужны для валидации

  //Можно добавить атрибут в HTML типа validation='true' или validation='false' и уже по нему исключаьть, так правильнее будет но лезть в HTML я не буду

  let input_array = $('.orderform_form_content input').not(

    '[name="year"], [name="mileage"], [name="complect"], [name="fueltype"], [name="note"], [name="marketing"]'

  );

  //Далее проходимся по массиву и смотрим есть ли value в input, если нет то закидываем value в функцию error, если нет то обнуляем функцией removeError

  $.each(input_array, function(key, value){

    //Собираем для каждого el его тип, смотря атрибут type

    let el_type = $(value).attr('type');    

    switch(el_type) {

      case 'text':

      case 'phone':

      //Ищем заголовок у поля

      text = $(value).parent().find('span').text();

      if(!$(value).val()){ error_input(value, text); }else{  

        //Тут же после проверяем наши суммы, ищем атрибут name чтобы совпадал с price_from и price_to

        let el_price = $(value).attr('name');

        switch(el_price) {

          //Если name price_from то обработаем его нам нужно получить тупо число

          case 'price_from':

            //Тут обрезаем ненужные пробелы и приводим цифру к единому виду

            price_from = $(value).val().replace(/\s/g, "");

            price_from = price_from.replace(/\D/g, '');

            //Тут проверяем условие нам нужно чтобы цифра была не меньше чем 1,5 млн

            if(price_from < 1500000){

              //Обрабатываем условие, если все таки меньше то кидаем в error

              error_input(value, 'price_from');

            }else{

              removeError(value);

            }

            if(!price_from){

              error_input(value, text);

            }

            break;

          //Если name price_to то обработаем его нам нужно получить тупо число  

          case 'price_to':  

            //Тут обрезаем ненужные пробелы и приводим цифру к единому виду

            price_to = $(value).val().replace(/\s/g, "");

            price_to = price_to.replace(/\D/g, '');

            //Тут мы смотри если price_to - price_from ниже нуля, то значит в price_to значение меньше чем в price_from, а так нельзя

            price_difference = Number(price_to) - Number(price_from);

            if(price_difference < 0){

              //Обрабатываем условие price_difference если меньге нуля то кидаем в error

              error_input(value, 'price_to');

            }else{

              removeError(value);

            }    

            if(!price_to){

              error_input(value, text);

            }

            break;

          //Если имя не price_to или price_from, значит простая обработка и тупо отправляем в error_input с дефолтным тектом

          default:

            removeError(value);

            break;  

        }

      };  

      //Завершаем case

      break;

    case 'radio':

      text = '<br>Укажите какой вид связи наиболее удобен для Вас';

      if(!$('input:radio').is(':checked')){ error_radio(value, text); }else{ removeError(value); }

      //Завершаем case  

      break;      

    }

  });  

  //В конце проверяем есть ли ошибки в window error, если их количество 0, то отправляем запрос

  if(window.ERROR.length == 0){ $('.errors').hide(); sendReq(); } else {

    //Если ошибки есть выведем их списком

    let text = 'Вы не заполнили поля: ';

    let count = window.ERRORTEXT.length - 1;

    $.each(window.ERRORTEXT, function(key, value){

      if(value != 'price_from' && value != 'price_to'){

        if(count != key){ text += value + ', '; }else{ text += value; }

      }else{

        if(value == 'price_from'){

          $('.errors').remove();

          $('.errors').hide();

          //Тут всплывашка над price_from

        }

        if(value == 'price_to'){

          $('.errors').remove();

          $('.errors').hide();          

          //Тут всплывашка над price_to

        }        

      }

    });

    $('.errors').html(text);

    $('.errors').show();

  }

});

//Функция обработки ошибки суммы

//Функция добавления ошибки валидации

function error_input(el, text) {

  //Добавляем текст span input чтобы отобразить его в списке того, что не заполнено

  window.ERROR.push(el);

  window.ERRORTEXT.push(text);

  //Выставляем в css анимацию в данный input

  $(el).css("animation", "0.5s errors ease-in");

    //Делаем плавное отключение анимации, через 500мс устанавливаем красную обводнку, потому что в конце анимации что выше она выключается

    //Теперь обводка будет висеть

    setTimeout(function () { $(el).css("border", "1px solid #fd5252"); }, 500);

    //Через 1000мс удаляем анимацию из css

    setTimeout(function () { $(el).css("animation", ""); }, 1000);

    //Через 10000мс отключаем обводку

    setTimeout(function () { $(el).css("border", ""); }, 10000);

    //Так как у нас прописан транзишн все это будет плавно ищезать

  //Тут мы скрываем значек загрузки, потому что обработка закончена  

  $("#mess_loading").hide();  

}

function error_radio(el, text) {

  //Добавляем текст span input чтобы отобразить его в списке того, что не заполнено

  window.ERROR.push(el);

  //Так как нам в принципе нужен только один текст ошибки на чекбоксы с выбором способа связи

  //Заносим его один раз. просто перед тем как занести проверить есть ли такой текст, если есть то ничего не делаем

  if(!window.ERRORTEXT.includes(text)){ window.ERRORTEXT.push(text); }

  //В radio почти так же как в input, только посвечиваем не обводку, а текст

  //Вешаем анимацию на текст

  $(el).parent().css("animation", "0.5s errors_text ease-in");

    //Делаем плавное отключение анимации, через 500мс устанавливаем красный текст, потому что в конце анимации что выше она выключается

    //Теперь текси будет висеть    

    setTimeout(function () { $(el).parent().css("color", "#fd5252"); }, 500);

    //Через 1000мс удаляем анимацию из css

    setTimeout(function () { $(el).parent().css("animation", ""); }, 1000);

    //Через 10000мс меняем текст на базовый

    setTimeout(function () { $(el).parent().css("color", "");  }, 10000);

    //Так как у нас прописан транзишн все это будет плавно ищезать

  //Тут мы скрываем значек загрузки, потому что обработка закончена  

  $("#mess_loading").hide();

}

  

//Функция отчистки валидации

function removeError(el) {

  //Собираем для каждого el его тип, смотря атрибут type

  let el_type = $(el).attr('type');  

  switch(el_type) {

  case 'text':

  case 'phone':

    //Просто у элемента удалем css с анимацией

    $(el).css("animation", "");

    //Удаляем ему обводку

    $(el).css("border", "");  

    //Завершаем case

    break;

  case 'radio':

  case 'checkbox':

    //Cтавим дефолтный цвет текста

    $(el).parent().css("color", "");

    //Завершаем case

    break;

  }  

}
```