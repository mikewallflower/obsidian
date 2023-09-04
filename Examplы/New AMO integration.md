###### tags: #examples 
###### links: [[Examples]]
___



```php
<?

Class Amocrm

{

    /* //Тут идут настройки отправления оповещения в Телеграмме, можно сменить бота и идентификатор человека/группы куда отправляем

    protected $telegramm_token = '1920317552:AAEcdlCY-NQcuJiWO9MJ7JMM6C2re1YuaAE';

    protected $telegramm_id = '-622622854'; */

  

    protected $amo_subdomen;

    protected $amo_secretkey;

    protected $amo_id;

    protected $amo_redirect_url = 'https://amocrmapi.ru/auth.php';

/*  protected $module_url = '/home/admin/web/kannam.group/public_html/core/integration/amocrm';

 */ //protected $module_url = '/home/admin/web/centr-polov.ru/public_html/upload/amo_integration/amonew/';

    protected $amo_funnel_id;

    protected $amo_responsible_user;

    protected $amo_refrash_token_check;

    protected $amo_refrash_token;

  

    protected $lead_city;

    protected $lead_ip;

    protected $lead_url;

    protected $lead_client_id;

    protected $lead_comment;

    protected $lead_name;

    protected $lead_email;

    protected $lead_phone;

    protected $lead_subject;

    protected $lead_summ;

    protected $lead_tags;

    protected $lead_utm_source;

    protected $lead_utm_medium;

    protected $lead_utm_campaign;

    protected $lead_utm_term;

    protected $lead_utm_content;

  

    protected $lead_utm_source_str;

    protected $lead_utm_medium_str;

    protected $lead_utm_campaign_str;

    protected $lead_utm_term_str;

    protected $lead_utm_content_str;

  

    protected $lead_city_str;

    protected $lead_ip_str;

    protected $lead_url_str;

    protected $lead_comment_str;

    protected $lead_client_id_str;

    protected $lead_phone_str;

    protected $lead_mail_str;

  

    protected $header;

    protected $access_token;

    protected $refresh_token_check;

    protected $sdelka_id;

  

    public function __construct($data){

        //Тут подключаем данные для отправки в АМО

        $this->amo_subdomen =            $data['AMO_SUBDOMEN'];

        $this->amo_secretkey =           $data['SECRET_KEY'];

        $this->amo_id =                  $data['API_ID'];  

        $this->amo_funnel_id =           $data['API_FUNNEL_ID'];

        $this->amo_responsible_user =    $data['API_RESPONSIBLE_USER'];

        $this->amo_refrash_token_check = $data['REFRASH_TOKEN_CHECK'];

        $this->amo_refrash_token =       $data['REFRASH_TOKEN'];

        //Тут подключаем данные лида

        $this->lead_city =               $data['CITY'];

        $this->lead_client_id =          $data['CLIENT_ID'];

        $this->lead_ip =                 $data['IP_LEAD'];

        $this->lead_url =                $data['URL_LEAD'];

        $this->lead_comment =            $data['MESSAGE'];

        $this->lead_name =               $data['NAME'];

        $this->lead_email =              $data['EMAIL'];

        $this->lead_phone =              $data['PHONE'];

        $this->lead_subject =            $data['SUBJECT'];

        $this->lead_summ =               $data['SUMM'];

        $this->lead_tags =               $data['TAGS'];

        $this->lead_utm_source =         $data['UTM_SOURCE'];

        $this->lead_utm_medium =         $data['UTM_MEDIUM'];

        $this->lead_utm_campaign =       $data['UTM_CAMPAIGN'];

        $this->lead_utm_term =           $data['UTM_TERM'];

        $this->lead_utm_content =        $data['UTM_CONTENT'];  

        //ID UTM-меток

        $this->lead_utm_source_str =     $data['UTM_SOURCE_STR'];

        $this->lead_utm_medium_str =     $data['UTM_MEDIUM_STR'];

        $this->lead_utm_campaign_str =   $data['UTM_CAMPAIGN_STR'];

        $this->lead_utm_term_str =       $data['UTM_TERM_STR'];

        $this->lead_utm_content_str =    $data['UTM_CONTENT_STR'];

        //ID полей АМО сделки

        $this->lead_city_str =           $data['CITY_NAME_ID'];

        $this->lead_ip_str =             $data['IP_LEAD_ID'];

        $this->lead_url_str =            $data['URL_LEAD_ID'];

        $this->lead_comment_str =        $data['COMMENT_ID'];

        $this->lead_client_id_str =      $data['ID_LEAD_ID'];

        $this->lead_phone_str =          $data['PHONE_ID'];

        $this->lead_mail_str =           $data['EMAIL_ID'];

    }

    public function control(){

        if($this->amo_refrash_token_check === 'Y'){

            return $this->getRefreshToken();

        }else{

            $token = $this->getAccessToken();

            if($token){

                return $this->sendLead();

            }

        }

    }

    //Получаем Access Token

    private function getAccessToken(){

        //Получаем текущий токен, который записан в файл

        $token_info = json_decode(file_get_contents($this->module_url . '/tokens/token_' . $this->amo_subdomen . '.json'));

        //Если мы не получаем ничего, то логируем и оповещаем

        if(!$token_info){

            $error = 'Отсутствует токен доступа для субдомена ' . $this->amo_subdomen . "\nЗапрос с сайта: " . $_SERVER['HTTP_HOST'] . "\nТелефон: " . $this->lead_phone;

            $this->sendTelegramAmoError($error);

        }

        //Формируем запрос а АМОСРМ

        $link = 'https://' . $this->amo_subdomen . '.amocrm.ru/oauth2/access_token';

        $data = array(

            'client_id' => $this->amo_id,

            'client_secret' => $this->amo_secretkey,

            'grant_type' => 'refresh_token',

            'refresh_token' => $token_info->refresh_token,

            'redirect_uri' => $this->amo_redirect_url

        );

        //Отправляем POST-запрос на получение Access Token

        $response = $this->post($link,$data,'token');

        //Обрабатываем ответ, если получили ошибку, то отправляем оповещение и логируем

        if($response['status']){

            $error = 'Ошибка получения токена для аккаунта ' . $this->amo_subdomen . "\nСтатус ошибки:   " . $response['status'] . "\nХинт:   " . $response['hint'] . "\nДетализация ошибки:   " . $response['detail'] . "\n\nТело запроса:\n" . serialize($data) . "\nТело ответа:\n" . serialize($response) . "\n\nСайт:\n" . $_SERVER['HTTP_REFERER'];

            $this->sendTelegramAmoError($error);

        }

        //Если все хорошо, подготавливаем новые данные к записи

        $access_token = $response['access_token'];

        $this->access_token = $access_token;

        $refresh_token = $response['refresh_token'];

        $token_type = $response['token_type'];

        $expires_in = $response['expires_in'];

        //Проверяем есть ли новый Access Token

        if($access_token){

            $data = array('access_token' => $access_token,'expires' => $expires_in,'refresh_token' => $refresh_token);

            //При получении перезаписываем в файл

            $this->writeJsonToFiles($data);

        }

        //В ответ отдаем Access Token

        return $response;

    }

    //Функция получения Refresh Token

    private function getRefreshToken() {

        //Получаем текущий токен, который записан в файл

        $token_info = json_decode(file_get_contents($this->module_url . '/tokens/token_' . $this->amo_subdomen . '.json'));

        //Формируем запрос а АМОСРМ

        $link = 'https://' . $this->amo_subdomen . '.amocrm.ru/oauth2/access_token';

        $data = array(

            'client_id' => $this->amo_id,

            'client_secret' => $this->amo_secretkey,

            'grant_type' => 'authorization_code',

            'code' => $this->amo_refrash_token,

            'redirect_uri' => $this->amo_redirect_url

        );

        //Отправляем POST-запрос на получение Access Token

        $response = $this->post($link,$data,'token');

        //Обрабатываем ответ, если получили ошибку, то отправляем оповещение и логируем

        if($response['status']){

            $error = 'Ошибка получения токена для аккаунта ' . $this->amo_subdomen . "\nСтатус ошибки:   " . $response['status'] . "\nХинт:   " . $response['hint'] . "\nДетализация ошибки:   " . $response['detail'] . "\n\nТело запроса:\n" . serialize($data) . "\nТело ответа:\n" . serialize($response) . "\n\nСайт:\n" . $_SERVER['HTTP_REFERER'];

            $this->sendTelegramAmoError($error);

        }

        //Если все хорошо, подготавливаем новые данные к записи

        $access_token = $response['access_token'];

        $this->access_token = $access_token;

        $refresh_token = $response['refresh_token'];

        $token_type = $response['token_type'];

        $expires_in = $response['expires_in'];

        //Проверяем есть ли новый Access Token

        if ($access_token){

            $data = array('access_token' => $access_token,'expires' => $expires_in,'refresh_token' => $refresh_token);

            //При получении перезаписываем в файл

            $this->writeJsonToFiles($data);

        }

        //В ответ отдаем Access Token

        return $access_token;

    }

    //Функция отправки лида в АМО СРМ

    private function sendLead(){

  

        $sdelka = array(

            'name' => $this->lead_phone . ', ' . $this->lead_city . ', ' . $this->lead_subject,

            'status_id' => $this->amo_funnel_id,

            'price' => $this->lead_summ,

            'responsible_user_id' => $this->amo_responsible_user,

            'tags'=> $this->lead_tags

        );

        $sdelka['custom_fields'][] = array('id' => $this->lead_city_str, 'values' => array(array('value' => $this->lead_city)));

        $sdelka['custom_fields'][] = array('id' => $this->lead_ip_str, 'values' => array(array('value' => $this->lead_ip)));

        $sdelka['custom_fields'][] = array('id' => $this->lead_url_str, 'values' => array(array('value' => $this->lead_url)));

        $sdelka['custom_fields'][] = array('id' => $this->lead_comment_str, 'values' => array(array('value' => $this->lead_comment)));

  

        if($this->lead_client_id){$sdelka['custom_fields'][] = array('id' => $this->lead_client_id_str,  'values' => array(array('value' => $this->lead_client_id)));}

  

        if($this->lead_utm_source  ){$sdelka['custom_fields'][] = array('id' => $this->lead_utm_source_str,  'values' => array(array('value' => $this->lead_utm_source)));}

        if($this->lead_utm_medium  ){$sdelka['custom_fields'][] = array('id' => $this->lead_utm_medium_str,  'values' => array(array('value' => $this->lead_utm_medium)));}

        if($this->lead_utm_campaign){$sdelka['custom_fields'][] = array('id' => $this->lead_utm_campaign_str,'values' => array(array('value' => $this->lead_utm_campaign)));}

        if($this->lead_utm_term    ){$sdelka['custom_fields'][] = array('id' => $this->lead_utm_term_str,    'values' => array(array('value' => $this->lead_utm_term)));}

        if($this->lead_utm_content ){$sdelka['custom_fields'][] = array('id' => $this->lead_utm_content_str, 'values' => array(array('value' => $this->lead_utm_content)));}

  

        $leads['request']['leads']['add'][] = $sdelka;

        $link = 'https://' . $this->amo_subdomen . '.amocrm.ru/private/api/v2/json/leads/set';

        //Отправляем POST-запрос, отправка сделки

        $response_sdelka = $this->post($link,$leads,'sendlead');

        session_start();

        //$_SESSION['id_amo'] = $response['response']['leads']['add'][0]['id'];

        $this->sdelka_id = $response_sdelka['response']['leads']['add'][0]['id'];

        if (!$_SESSION['code'] === 200){

            $error = "Ошибка при отправке лида в AMO\nКод ошибки: " . $code . "\nТело ответа: " . serialize(curl_getinfo($curl));

            $this->sendTelegramAmoError($error);

        }

        if ($_SESSION['code'] === 200){

            if(!$this->lead_name){$this->lead_name = 'Имя неизвестно';}

            $contact = array(

                'name' => $this->lead_name,

                'linked_leads_id' => array($this->sdelka_id),

                'responsible_user_id' => $this->amo_responsible_user

            );

        if($this->lead_client_id){$sdelka['custom_fields'][] = array('id' => $this->lead_client_id_str,  'values' => array(array('value' => $this->lead_client_id)));}

  

        if($this->lead_phone){$contact['custom_fields'][] = array('id' => $this->lead_phone_str, 'values' => array(array('value' => $this->lead_phone, 'enum' => 'WORK')));}

        if($this->lead_email){$contact['custom_fields'][] = array('id' => $this->lead_mail_str,'values' => array(array('value' => $this->lead_email, 'enum' => 'WORK')));}

  

        $set['request']['contacts']['add'][] = $contact;

        $link ='https://' . $this->amo_subdomen . '.amocrm.ru/private/api/v2/json/contacts/set';

        //Отправляем POST-запрос, отправка контакта

        $responses = $this->post($link,$set,'sendlead');

            if($this->lead_comment){

                $notes['request']['notes']['add'] = array(

                    array(

                        'element_id' => $this->sdelka_id,

                        'element_type' => 2,

                        'note_type' => 4,

                        'text' => $this->lead_comment,

                        'responsible_user_id' => $this->amo_responsible_user

                    )

                );

                $link = 'https://' . $this->amo_subdomen  . '.amocrm.ru/private/api/v2/json/notes/set';

                //Отправляем POST-запрос, отправка комментария

                $response = $this->post($link,$notes,'sendlead');

            }

        return $responses;

        }else{

            //Логируем ощибку, но вообще тут можно не писать а сразу куда либо отправлять. Пока лог отключю.

        }

    }

    //Отправляем POST-запрос в АМО

    private function post($link,$data,$type){

        $curl = curl_init();

        curl_setopt($curl,CURLOPT_RETURNTRANSFER,true);

        curl_setopt($curl,CURLOPT_USERAGENT,'amoCRM-API-client/1.0');

        curl_setopt($curl,CURLOPT_URL,$link);

        curl_setopt($curl,CURLOPT_HTTPHEADER,array('Content-Type: application/json'));

        curl_setopt($curl,CURLOPT_HEADER,false);

        curl_setopt($curl,CURLOPT_CUSTOMREQUEST,'POST');

        curl_setopt($curl,CURLOPT_POSTFIELDS,json_encode($data));

        if($type === 'sendlead'){

            $this->header = array('Authorization: Bearer ' . $this->access_token,"Content-Type: application/json");

            curl_setopt($curl,CURLOPT_HTTPHEADER, $this->header);

            curl_setopt($curl,CURLOPT_SSL_VERIFYPEER,0);

            curl_setopt($curl,CURLOPT_SSL_VERIFYHOST,0);

            session_start();

        }

        if($type === 'token'){

            curl_setopt($curl,CURLOPT_SSL_VERIFYPEER, 1);

            curl_setopt($curl,CURLOPT_SSL_VERIFYHOST, 2);          

        }

        $out = curl_exec($curl);

        if($type === 'sendlead'){

            $_SESSION['code'] = curl_getinfo($curl,CURLINFO_HTTP_CODE);

        }

        $response = json_decode($out, true);

        return $response;

    }

    //Пишем токен в файл

    private function writeJsonToFiles($data){

        $str = str_replace('"', '', $this->amo_subdomen);

        $json = json_encode($data);

        $file = fopen($this->module_url . '/tokens/token_' . $str . '.json','w+');

        fwrite($file, $json);

        fclose($file);

    }

    //Отправляем оповещение в телеграмм

    private function sendTelegramAmoError($error){

        $ch = curl_init();

        curl_setopt_array(

            $ch,

            array(

                CURLOPT_URL => 'https://api.telegram.org/bot' . $this->telegramm_token . '/sendMessage',

                CURLOPT_POST => TRUE,

                CURLOPT_RETURNTRANSFER => TRUE,

                CURLOPT_TIMEOUT => 10,

                CURLOPT_POSTFIELDS => array(

                    'chat_id' => $this->telegramm_id,

                    'text' => $error,

                ),

            )

        );

        curl_exec($ch);

    }

}

?>
```


![[Pasted image 20230809100223.png]]