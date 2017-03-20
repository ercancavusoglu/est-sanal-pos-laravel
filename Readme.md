# Teb Sanal Pos Entegrasyon Modülü PHP Laravel

Laravel tabanlı yazdığım est tabanlı sanal pos kütüphanesi. 3D Secure modüllü olarak hazırlanmıştır.

Entegrasyon veya destek için issue açabilir, `ercancavusoglu@yandex.com.tr` adresinden veya [@devredisibirak](http://twitter.com/devredisibirak) twitter hesabımdan bana ulaşabilirsiniz.

## Kurulum

	namespace App\Http\Controllers;
	use Illuminate\Http\Request;
	use App\Http\Requests;
	use App\Http\Controllers\Controller;

	//Library dahil ediliyor
	use App\Libraries\SanalPos\est3dModel;
	//Library dahil ediliyor

	class TestController extends Controller
	{
	    public function __construct()
	    {
	        parent::__construct();       
	        $this->est3dModel=new est3dModel();        
	    }
	    public function postProcess(Request $request)
	    {
	        $price = 100;
	        $orders_no = "SİP NO";
	        $webpos_bank['cc_owner'] = $request->firstname;
	        $webpos_bank['cc_number'] = $number;
	        $webpos_bank['cc_cvv2'] = $request->cvv;
	        $webpos_bank['cc_expire_date_month'] = $month;
	        $webpos_bank['cc_expire_date_year'] = $request->expiryYear;
	        $webpos_bank['cc_type'] = $request->card_type;
	        $webpos_bank["bank_id"] = $request->webpos_bank_id;
	        $webpos_bank['customer_ip'] = ip();
	        $webpos_bank['instalment'] = $request->installment;
	        $webpos_bank['success_url'] = url('test/callback'); //bank will return here if payment successfully finishes
	        $webpos_bank['fail_url'] = url('test/callback'); //bank will return here if payment fails;
	        $webpos_bank['order_id'] = $orders_no;
	        $webpos_bank['total'] = $price;
	        $webpos_bank['mode'] = "live";
	        $webpos_bank['order_info'] = "";
	        $webpos_bank['products'] = "";
	        $method_response = $this->est3dModel->methodResponse($webpos_bank);
	       
	        if (isset($method_response['form'])) {
	            $json['form'] = $method_response['form'];
	        } else if (isset($method_response['error'])) {
	            $message = (isset($method_response['message'])) ? $method_response['message'] : '';           
	            $json['error'] = $method_response['error'].$message;
	        }
	        return json_encode($json);
	    }
	    public function anyCallback(Request $req)
	    {
	        $bank_response = $req->all();
	        $webpos_bank=[];//Ekstra değer göndermek için
	        
	        //Para aktarımını başlatıyoruz
	        $method_response = $this->est3dModel->bankResponse($bank_response, $webpos_bank);
	        
	        $orders_no = $bank_response["oid"];

	        if(isset($bank_response["taksit"]) and !empty($bank_response["taksit"])){
	            $instalment = $bank_response["taksit"];
	            $instalment=($instalment=="") ? 0 : $instalment;
	        }else{
	            $instalment = 0;
	        }        
	        if ($method_response['result'] == 1) {
	            $message = $method_response['message'] . $banka;
	            $price=$bank_response["amount"];//Çekilen tutar- Kontrol için kullanılabilir
	            
	            //Siparişi kaydet
	            //Mail gönder
	            //Sepeti boşalt 
	            
	            $data['continue'] = url('test/success/' . $orders_no);
	            $data['message'] = $method_response['message'];
	            return Redirect::to("test/success/" . $orders_no)->with("data", $data);            
	        } else {            
	            $message = $method_response['message'];                        
	            $price = $bank_response["amount"];//Log için tutulabilir            
	            return Redirect::to("kart-sayfasi-buraya")->with("error", $message);            
	        }
	    }
	}