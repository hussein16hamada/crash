


class FloatingWindowService : Service() {
    lateinit var azkar :Array<String>
    private lateinit var windowManager: WindowManager
    private lateinit var linearLayout: LinearLayout
    private lateinit var windowManagerLayoutParams: WindowManager.LayoutParams
    lateinit var textView :TextView
    lateinit var tinyDB: TinyDB
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        Log.d("sdsdsdsdsdsds", "onStartCommand: ")

        if (intent?.getStringExtra("t")=="t"){
            NewAlarmActivity.setAlarm(this,tinyDB.getInt("timeForRandomAzkar"))
            Log.d("sdsdsdsdsdsds", "intent: ")

            applicationContext.startActivity(Intent(this,HomePageActivity::class.java))
        }
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) startForeground("أذكار")
        else startForeground(1, Notification())
        return START_STICKY
    }
    override fun onBind(p0: Intent?): IBinder? {
        return null
    }

    override fun onCreate() {
        super.onCreate()
        tinyDB= TinyDB(this)
        azkar=resources.getStringArray(R.array.randomAzkar)
        Log.d("sdsdsdsdsdsds", "onCreate: ")


//        createLinearLayout()
//        createCloseButton()
        textView = TextView(this)
        linearLayout= LinearLayout(this)
        linearLayout.addView(textView)
        textView.setBackgroundResource(R.drawable.roundedbar)
        try {
            textView.text = azkar.random()

        }catch (e:Exception){
            Log.d("eeeeeeeeee", e.localizedMessage)
        }
        textView.setTextColor(Color.rgb(0, 130, 200))
        textView.textSize = 17.0f
        textView.setPadding(22, 16, 25, 16)
        setupWindowManager()
        windowManager.addView(linearLayout, windowManagerLayoutParams)

        linearLayout.setOnClickListener {

            textView.fadeVisibility(View.GONE,900)
//            windowManager.removeView(linearLayout)
            NewAlarmActivity.setAlarm(this,tinyDB.getInt("timeForRandomAzkar"))

            stopSelf()


//            stopSelf()


        }
//        setupLayoutMovement()


    }

    override fun onDestroy() {
        super.onDestroy()
       linearLayout.post { object :Runnable{
           override fun run() {
               windowManager.removeView(linearLayout)
           }
       } }
    }
    fun View.fadeVisibility(visibility: Int, duration: Long = 400) {
        val transition: Transition = Slide(Gravity.RIGHT)
        transition.duration = duration
        transition.addTarget(this)
        TransitionManager.beginDelayedTransition(this.parent as ViewGroup, transition)
        this.visibility = visibility
    }
    private fun createCloseButton() {
        val closeButton = Button(this)
        closeButton.text = "Close"
        closeButton.layoutParams = ViewGroup.LayoutParams(
            ViewGroup.LayoutParams.MATCH_PARENT,
            ViewGroup.LayoutParams.WRAP_CONTENT
        )
        linearLayout.addView(closeButton)

        closeButton.setOnClickListener {
            windowManager.removeView(linearLayout)
            stopSelf()
            val transition: Transition = Fade()
            transition.setDuration(600)
            transition.addTarget(closeButton)

            TransitionManager.beginDelayedTransition(linearLayout, transition)

        }
    }


    private fun startForeground(type: String) {
        val channelId =
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                createNotificationChannel("FloatingService", "My floating Background Service")
            } else {
                // If earlier version channel ID is not used
                // https://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html#NotificationCompat.Builder(android.content.Context)
                ""

            }


        val notificationBuilder = NotificationCompat.Builder(this, channelId)
        val notification = notificationBuilder.setOngoing(true)
            .setSmallIcon(R.mipmap.ic_launcher)
            .setPriority(NotificationCompat.PRIORITY_MIN)
            .setContentTitle("المنبة")
            .setContentText(type)
            .setCategory(Notification.CATEGORY_SERVICE)
            .build()
        startForeground(110, notification)

    }



        @RequiresApi(Build.VERSION_CODES.O)
        fun createNotificationChannel(channelId: String, channelName: String): String {
            val chan = NotificationChannel(
                channelId,
                channelName, NotificationManager.IMPORTANCE_NONE
            )
            chan.lightColor = Color.BLUE
            chan.lockscreenVisibility = Notification.VISIBILITY_PRIVATE
            val service = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
            service.createNotificationChannel(chan)
            return channelId
        }

    private fun setupWindowManager() {
        val LAYOUT_FLAG: Int
        LAYOUT_FLAG = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY
        } else {
            WindowManager.LayoutParams.TYPE_PHONE
        }
        windowManager = getSystemService(WINDOW_SERVICE) as WindowManager
        windowManagerLayoutParams = WindowManager.LayoutParams(
            WindowManager.LayoutParams.WRAP_CONTENT,
            WindowManager.LayoutParams.WRAP_CONTENT,
            LAYOUT_FLAG,
            WindowManager.LayoutParams.FLAG_LAYOUT_NO_LIMITS or
                    WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE or
                    WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL or
                    WindowManager.LayoutParams.FLAG_WATCH_OUTSIDE_TOUCH,
                    PixelFormat.TRANSLUCENT
        )
        windowManagerLayoutParams.gravity = Gravity.TOP or Gravity.RIGHT
        windowManagerLayoutParams.x = 0
        windowManagerLayoutParams.y = 100

//        windowManagerLayoutParams.gravity = Gravity.CENTER or Gravity.CENTER
    }


}
