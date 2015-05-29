---
title: Android SQLite数据库初步  
date: 2015-5-6 22:45
layout: post
category: technology
---
SQLite是一个嵌入式的轻量数据库，适用于资源有限的设备上。  
SQLite教程：[SQLite](http://www.w3cschool.cc/sqlite/sqlite-tutorial.html "SQLite")  
Android中提供SQLiteDatabase类代表一个数据库，在程序中只要获得代表指定数据库的SQLiteDatabase对象后，就可以通过此对象来操作数据库了。  
在实际使用中，我们一般是实现一个继承自SQLiteOpenHelper的类来获取数据库操作对象。通过重写SQLiteOpenHelper的`onCreate`及`onUpgrate`方法来定义自己所需的获取更新数据库对象的操作。  
示例代码如下：  
{% highlight java %}  
    public class DatabaseHelper extends SQLiteOpenHelper {
		// 建表的SQL语句
	    private static final String CREATE_TABLE_GOAL = "CREATE TABLE " +
	            "Goal(" +
	            "_id integer primary key autoincrement," +
	            "name text not null," +
	            "date text not null)";
	    private static final String DATABASE_NAME = "MileStoneDatabase";
	    private static final int DATABASE_VERSION = 1;
	    public DatabaseHelper(Context context) {
	        super(context, DATABASE_NAME, null, DATABASE_VERSION);
	    }
	
	    @Override
	    public void onCreate(SQLiteDatabase db) {
	        db.execSQL(CREATE_TABLE_GOAL);
	    }
	
	    @Override
	    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
	
	    }
	}
{% endhighlight %}  
之后，就可以通过自定义一个DatabaseAdapter类，包含成员DatabaseHelper及SQLiteDatabase，这样就能封装数据库操作，示例代码如下：  
{% highlight java %}  
	public class DatabaseAdapter {
		// 定义Table name
	    private static final String TABLE_GOAL = "Goal";

	    private DatabaseHelper mDbHelper;
	    private SQLiteDatabase mDb;
	
	    public final Context mCtx;
	
	    public DatabaseAdapter(Context ctx) {
	        this.mCtx = ctx;
	    }
	
	    public DatabaseAdapter openWrite() {
	        mDbHelper = new DatabaseHelper(mCtx);
	        mDb = mDbHelper.getWritableDatabase();
	        return this;
	    }

	    public DatabaseAdapter openRead() {
	        mDbHelper = new DatabaseHelper(mCtx);
	        mDb = mDbHelper.getReadableDatabase();
	        return this;
	    }
	
	    public void close() {
	        mDb.close();
	    }
		//…… 其他数据库操作
	}

{% endhighlight %}  
如果具有大量的数据，还是需要使用服务器，SQLite只是一个轻量的数据库，无法替代服务器。
