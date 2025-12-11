对NotePad笔记的功能添加时间戳和笔记查询功能（不区分大小写）
附加功能美化了笔记展示与编辑页面
新增了笔记编辑页面更换背景色功能（在右上角的多功能框中选择）
拓展附加功能：
1.添加时间戳和字数：在notelist_item.xml使用testview
<TextView
            android:id="@+id/note_timestamp"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="2 seconds ago"
            android:textSize="12sp"
            android:textColor="#999999"
            android:gravity="start"
            android:fontFamily="sans-serif-light" />

        <TextView
            android:id="@+id/note_word_count"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="0 字"
            android:textSize="12sp"
            android:textColor="#999999"
            android:gravity="start"
            android:fontFamily="sans-serif-light" />
然后在适配器（比如 NoteListAdapter）中设置时间戳的显示：
public class NoteListAdapter extends ArrayAdapter<Note> {
    // ...

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        // ...
        TextView timestampTextView = (TextView) convertView.findViewById(R.id.note_timestamp);
        timestampTextView.setText(formatTimestamp(notes.get(position).getTimestamp()));
        // ...
        return convertView;
    }

    private String formatTimestamp(long timestamp) {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault());
        return sdf.format(new Date(timestamp));
    }
}
<img width="526" height="184" alt="image" src="https://github.com/user-attachments/assets/3c808800-7954-4697-9ec3-38478291fc71" />
效果图
然后在 NoteEditorActivity 中设置和获取时间戳：
public class NoteEditor extends AppCompatActivity {
    // ...

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_editor);

        TextView timestampTextView = (TextView) findViewById(R.id.note_timestamp_editor);
        Note note = getIntent().getParcelableExtra("note");
        if (note != null) {
            timestampTextView.setText(formatTimestamp(note.getTimestamp()));
        } else {
            timestampTextView.setText(formatTimestamp(System.currentTimeMillis()));
        }
        // ...
    }

    // ...
}
2.查询功能添加：
activity_note_list.xml（布局文件修改）在笔记列表的布局文件中添加一个搜索框。可以使用SearchView组件。
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <androidx.appcompat.widget.SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

</LinearLayout>
NoteList.java（Activity类修改）在对应的Activity中处理搜索逻辑
package com.example.android.notepad;

import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.SearchView;

import android.os.Bundle;
import android.widget.ArrayAdapter;
import android.widget.ListView;

import java.util.ArrayList;
import java.util.List;

public class NoteList extends AppCompatActivity {

    private ListView listView;
    private SearchView searchView;
    private ArrayAdapter<String> adapter;
    private List<String> noteTitles = new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_note_list);

        listView = findViewById(R.id.list_view);
        searchView = findViewById(R.id.search_view);

        noteTitles.add("Note 1");
        noteTitles.add("Note 2");
        noteTitles.add("Another Note");

        adapter = new ArrayAdapter<>(this, R.layout.noteslist_item, noteTitles);
        listView.setAdapter(adapter);

        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String query) {
                return false;
            }

            @Override
            public boolean onQueryTextChange(String newText) {
                adapter.getFilter().filter(newText);
                return false;
            }
        });
    }
}
noteslist_item.xml（列表项布局文件修改）
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/note_title"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="16dp"
    android:textSize="18sp" />
数据源相关（如果是从数据库获取数据）
NotePadProvider.java（Content Provider类）
在Content Provider中添加查询方法，例如：
@Override
public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
    SQLiteDatabase db = mDbHelper.getReadableDatabase();
    SQLiteQueryBuilder builder = new SQLiteQueryBuilder();
    builder.setTables(NotePad.Notes.TABLE_NAME);

    if (selection != null) {
        builder.appendWhere(selection);
    }

    Cursor cursor = builder.query(db, projection, selection, selectionArgs, null, null, sortOrder);
    cursor.setNotificationUri(getContext().getContentResolver(), uri);
    return cursor;
}
在NoteList.java中使用Content Provider查询数据
Cursor cursor = getContentResolver().query(
        NotePad.Notes.CONTENT_URI,
        new String[]{NotePad.Notes.COLUMN_NAME_TITLE},
        null,
        null,
        null
);
if (cursor != null && cursor.moveToFirst()) {
    do {
        String title = cursor.getString(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE));
        noteTitles.add(title);
    } while (cursor.moveToNext());
    cursor.close();
}
searchable文件
<?xml version="1.0" encoding="utf-8"?>
<searchable xmlns:android="http://schemas.android.com/apk/res/android"
    android:label="@string/app_name"
    android:hint="@string/search_hint"
    android:includeInGlobalSearch="false"
    android:searchSuggestAuthority="com.example.android.notepad.NotePadProvider"
    android:searchSuggestSelection=" ?" />
<img width="546" height="887" alt="image" src="https://github.com/user-attachments/assets/547c5797-48d5-4a2a-94f1-c54fb3fe92c0" />
这是界面图
<img width="581" height="684" alt="image" src="https://github.com/user-attachments/assets/62ea26fa-75cf-4335-9f98-5043e448db33" />
这是搜索效果图
3.页面美化：
由于项目版本过低，很多依赖无法解析，考虑不添加依赖的美化方案：
在不引入任何第三方库的前提下，从以下几个方面优化：调整内边距和边距，让编辑区域更舒适，优化文字样式（字号、行距、颜色），设置更美观的背景，改进滚动条样式，添加适当的边距和布局容器
美化后效果如下：
<img width="521" height="981" alt="image" src="https://github.com/user-attachments/assets/0d15acda-ae2c-4356-ba95-a045bf8b22ba" />
4.新增笔记编辑页面更换背景色功能：
通过以下步骤实现：
在布局中为 EditText 设置一个背景色属性，可以动态更改，在 Activity/Fragment 中定义几种背景色，添加一个简单的颜色选择机制，点击后动态更改 EditText 的背景色，新增colors.xml文件记录颜色，然后修改布局增加修改背景色的按钮，最后应用到activityjava文件当中即可实现
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/menu_save"
        android:icon="@drawable/ic_menu_save"
        android:alphabeticShortcut='s'
        android:title="@string/menu_save"
        android:showAsAction="ifRoom|withText" />
    <item android:id="@+id/menu_revert"
        android:icon="@drawable/ic_menu_revert"
        android:title="@string/menu_revert" />
    <item android:id="@+id/menu_delete"
        android:icon="@drawable/ic_menu_delete"
        android:title="@string/menu_delete"
        android:showAsAction="ifRoom|withText" />
    <item android:id="@+id/menu_change_bg"
        android:title="更换背景色" />
</menu>
效果图和选择框如下：
<img width="549" height="1053" alt="image" src="https://github.com/user-attachments/assets/befafff0-4a7f-4409-ba59-53a2afd6b34d" />
<img width="515" height="1052" alt="image" src="https://github.com/user-attachments/assets/a7bceaba-de5a-4fd0-ba9c-3c0c5df74fe8" />
5，新增笔记排序功能：
包括了按字数排序，按首字母排序，按日期（时间戳）排序
第一步：修改菜单文件添加排序选项
第二步：xml文件添加字符串资源
<resources>
    <string name="app_name">NotePad</string>
    <string name="live_folder_name">Notes</string>

    <string name="title_edit_title">Note title:</string>
    <string name="title_create">New note</string>
    <string name="title_edit">Edit: %1$s</string>
    <string name="title_notes_list">Notes</string>

    <string name="menu_add">New note</string>
    <string name="menu_save">Save</string>
    <string name="menu_delete">Delete</string>
    <string name="menu_open">Open</string>
    <string name="menu_revert">Revert changes</string>
    <string name="menu_copy">Copy</string>
    <string name="menu_paste">Paste</string>

    <string name="button_ok">OK</string>
    <string name="text_title">Title:</string>

    <string name="resolve_edit">Edit note</string>
    <string name="resolve_title">Edit title</string>

    <string name="error_title">Error</string>
    <string name="error_message">Error loading note</string>
    <string name="nothing_to_save">There is nothing to save</string>
    <string name="menu_search">搜索</string>
    <string name="search_hint">搜索笔记...</string>
    <string name="note_page_description">笔记编辑页面，包含标题和编辑区域</string>
    <string name="note_title_description">笔记标题栏</string>
    <string name="note_editor_description">笔记编辑区域，可以在这里输入和编辑您的笔记内容</string>
    <string name="note_editor_hint">在此输入笔记内容...</string>
    <string name="menu_sort">排序</string>
    <string name="sort_by_title">按标题排序</string>
    <string name="sort_by_word_count">按字数排序</string>
    <string name="sort_by_date">按日期排序</string>
    <string name="sort_order_ascending">升序</string>
    <string name="sort_order_descending">降序</string>
第三步：修改 NotesList.java 添加排序功能
第四步：在 Provider 中确保支持排序
<item android:id="@+id/menu_sort"
        android:title="排序"
        android:alphabeticShortcut='o'
        android:showAsAction="never">
        <menu>
            <item android:id="@+id/menu_sort_title"
                android:title="按标题排序" />
            <item android:id="@+id/menu_sort_word_count"
                android:title="按字数排序" />
            <item android:id="@+id/menu_sort_date"
                android:title="按日期排序" />
        </menu>
    </item>
下面为选择框和排序效果
<img width="546" height="1064" alt="image" src="https://github.com/user-attachments/assets/01358396-a0ea-40f7-acad-5563405c8c1a" />
<img width="534" height="864" alt="image" src="https://github.com/user-attachments/assets/9552228c-ba47-425d-8ac7-a97bc6999cde" />

