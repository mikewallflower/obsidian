###### tags: #examples
###### links: [[Examples]]
___



```php
<?

// Метод control класса ожидает массив c полями $list_id, $name, $email

//Подключение интеграции на рассылку к Unisender.com

/*

if ($_POST['user_mail']) {

    $sender = new UnisenderApi();

    $data = array(

        'list_id' => '393';                //id списка рассылки на Unisender

        'name' => $_POST['user_name'];

        'email' => $_POST['user_mail'];

    );

    $sender -> control($data);

}

*/



class UnisenderApi {


    private $api_key = '61jiwzk44i8okzgfi48wthsge1popew65datqebe';


    public function control($data){

        $link = 'subscribe';

        $string = 'api_key=' . $this->apikey . '&list_ids=' . $this->$data['list_id'] . '&fields[Name]=' . $data['name'] . '&fields[email]=' . $data['email'] . '&double_optin=3&overwrite=0';

        return $this->sendGet($link,$string);

        return $this->subscribe($params);

    }

  

    private function subscribe($params) {

        return $this->get($params);

    }

  

    private function get($params)

    {

        $ch = curl_init("https://api.unisender.com/ru/api/subscribe?" . http_build_query($params));  

        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

        curl_setopt($ch, CURLOPT_CUSTOMREQUEST, "GET");

        $answer = curl_exec($ch);

        curl_close($ch);

        return json_decode($answer);

    }

}

  

//Рабочая рассылка

  

class UnisenderApi{

  

    protected $apikey = '61jiwzk44i8okzgfi48wthsge1popew65datqebe';

  

    public function subscribe($list_id, $name, $email){

        $link = 'subscribe';

        $string = 'api_key=' . $this->apikey . '&list_ids=' . $list_id . '&fields[Name]=' . $name . '&fields[email]=' . $email . '&double_optin=3&overwrite=0';

        return $this->sendGet($link,$string);

    }

  

    public function getList(){

        $link = 'getLists';

        $string = 'api_key=' . $this->apikey;

        return $this->sendGet($link,$string);

    }

  

    private function sendGet($link,$string){

        $ch = curl_init('https://api.unisender.com/ru/api/' . $link . '?format=json&' . $string);

        curl_setopt($ch, CURLOPT_CUSTOMREQUEST, "GET");

        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);

        $data_derty = curl_exec($ch);

        $data_ = json_decode($data_derty);

        return $data_;

    }

}

?>
```