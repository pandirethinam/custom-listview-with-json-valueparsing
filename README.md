# custom-listview-with-json-valueparsing
customlistview
public class MainActivity extends Activity {

    private static final String url = "http://api.androidhive.info/json/movies.json";
    String responseFromServer;
    ProgressDialog progressDialog;
    CinemaItemBean cineItemBean;
    ListView cinemaList;
    ArrayList<CinemaItemBean> resultList=new ArrayList<>();


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        new DownloadMovieDetails().execute();

    }

    public class DownloadMovieDetails extends AsyncTask<String ,Void,String>{

        @Override
        protected String doInBackground(String... params) {
            try{
                HttpClient client=new DefaultHttpClient();
                HttpGet getRequest=new HttpGet(url);//Forming the request.
                HttpResponse response=client.execute(getRequest);
                HttpEntity responseEntity=response.getEntity();
                responseFromServer= EntityUtils.toString(responseEntity);



            }catch (Exception e){
                e.printStackTrace();
            }
            return responseFromServer;
        }
        @Override
        protected void onPreExecute() {
            super.onPreExecute();
            progressDialog=new ProgressDialog(MainActivity.this);
            progressDialog.setMessage("Loading Please wait....");
            progressDialog.setCancelable(false);
            progressDialog.show();
        }
        @Override
        protected void onPostExecute(String response) {
            super.onPostExecute(response);
            if(progressDialog.isShowing()){
                progressDialog.dismiss();
            }
            try{
                JSONArray responseArray=new JSONArray(response);
                for (int i=0;i<responseArray.length();i++){
                    JSONObject itemObj=responseArray.getJSONObject(i);
                    cineItemBean=new CinemaItemBean();
                    cineItemBean.setTitle(itemObj.getString("title"));
                    cineItemBean.setYear(itemObj.getInt("releaseYear"));
                    cineItemBean.setRating(itemObj.getDouble("rating"));
                    cineItemBean.setThumbnailUrl(itemObj.getString("image"));

                    ArrayList<String> genreList=new ArrayList<>();
                    JSONArray genreArray=itemObj.getJSONArray("genre");
                    for (int j=0;j<genreArray.length();j++){
                        Log.e("NAme",genreArray.getString(j));
                        genreList.add(genreArray.getString(j));
                    }
                    cineItemBean.setGenre(genreList);
                    resultList.add(cineItemBean);
                }
                cinemaList= (ListView) findViewById(R.id.cinemaList);
                CinemaAdapter adapter=new CinemaAdapter(MainActivity.this,R.layout.cinema_list_item,resultList);
                cinemaList.setAdapter(adapter);

                cinemaList.setOnItemClickListener(new AdapterView.OnItemClickListener() {
                    @Override
                    public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                        CinemaItemBean itemBean=resultList.get(position);
                        String imageUrl=itemBean.getThumbnailUrl();
                        String title=itemBean.getTitle();
                        int releaseYear=itemBean.getYear();
                        ArrayList<String> genreList=itemBean.getGenre();
                        Intent detailIntent=new Intent(MainActivity.this,DetailActivity.class);
                        detailIntent.putExtra("ImageUrl",imageUrl);
                        detailIntent.putExtra("Title",title);
                        detailIntent.putExtra("ReleaseYear",releaseYear);
                        String genre="";
                        for (int i=0;i<genreList.size();i++){
                            genre=genre+genreList.get(i)+",";
                        }
                        detailIntent.putExtra("Genre",genre);
                        startActivity(detailIntent);




                    }
                });


            }catch (Exception e){
                e.printStackTrace();
            }
            Log.e("Response",response);
        }
    }



}
