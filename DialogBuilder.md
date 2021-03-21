# DialogBuilder

DialogBuilder.java

```java
package com.pateo.haima.carwallet.view;

import android.app.AlertDialog;
import android.content.Context;
import android.support.annotation.StringRes;
import android.view.LayoutInflater;
import android.view.View;
import android.view.Window;
import android.widget.Button;
import android.widget.TextView;

import com.pateo.haima.carwallet.R;

/**
 * 类  名： DialogBuilder <br/>
 * 包  名： com.pateo.haima.carwallet.view <br/>
 * 描  述：  <br/>
 * 作  者： fengtao <br/>
 * 日  期： 2019-05-23 17:28 <br/>
 */
public class DialogBuilder {

    public static DialogBuilder with(Context context) {
        return new DialogBuilder(context);
    }

    public static final int WHICH_BUTTON_POSITIVE = 0x001;
    public static final int WHICH_BUTTON_NEGATIVE = 0x002;

    private AlertDialog.Builder mBuilder;
    private AlertDialog mDialog;

    private int mWidth = 840, mHeight = 420;

    private View mContentView;

    private boolean mIfPositiveButtonShown = false, mIfNegativeButtonShown = false;

    private DialogBuilder(Context context) {
        mBuilder = new AlertDialog.Builder(context);
        mContentView = LayoutInflater.from(context).inflate(R.layout.layout_dialog, null);
        mBuilder.setView(mContentView);
    }

    public DialogBuilder size(int width, int height) {
        mWidth = width;
        mHeight = height;
        return this;
    }

    public DialogBuilder message(@StringRes int message) {
        TextView tvMessage = mContentView.findViewById(R.id.tv_message);
        tvMessage.setText(message);
        return this;
    }

    public DialogBuilder message(String message) {
        TextView tvMessage = mContentView.findViewById(R.id.tv_message);
        tvMessage.setText(message);
        return this;
    }

    public DialogBuilder buttonPositive(CharSequence text, OnClickListener listener) {
        Button btnPositive = mContentView.findViewById(R.id.btn_positive);
        btnPositive.setVisibility(View.VISIBLE);
        btnPositive.setText(text);
        btnPositive.setOnClickListener(v -> {
            if (listener != null && mDialog != null) {
                listener.onClick(mDialog, WHICH_BUTTON_POSITIVE);
            }
        });
        mIfPositiveButtonShown = true;
        return this;
    }

    public DialogBuilder buttonPositive(@StringRes int text, OnClickListener listener) {
        Button btnPositive = mContentView.findViewById(R.id.btn_positive);
        btnPositive.setVisibility(View.VISIBLE);
        btnPositive.setText(text);
        btnPositive.setOnClickListener(v -> {
            if (listener != null && mDialog != null) {
                listener.onClick(mDialog, WHICH_BUTTON_POSITIVE);
            }
        });
        mIfPositiveButtonShown = true;
        return this;
    }

    public DialogBuilder buttonNegative(CharSequence text, OnClickListener listener) {
        Button btnNegative = mContentView.findViewById(R.id.btn_negative);
        btnNegative.setVisibility(View.VISIBLE);
        btnNegative.setText(text);
        btnNegative.setOnClickListener(v -> {
            if (listener != null && mDialog != null) {
                listener.onClick(mDialog, WHICH_BUTTON_NEGATIVE);
            }
        });
        mIfNegativeButtonShown = true;
        return this;
    }

    public DialogBuilder buttonNegative(@StringRes int text, OnClickListener listener) {
        Button btnNegative = mContentView.findViewById(R.id.btn_negative);
        btnNegative.setVisibility(View.VISIBLE);
        btnNegative.setText(text);
        btnNegative.setOnClickListener(v -> {
            if (listener != null && mDialog != null) {
                listener.onClick(mDialog, WHICH_BUTTON_NEGATIVE);
            }
        });
        mIfNegativeButtonShown = true;
        return this;
    }

    public AlertDialog show() {
        setLineShowOrHide();
        mDialog = mBuilder.create();
        Window window = mDialog.getWindow();
        if (window != null) {
            window.setBackgroundDrawableResource(android.R.color.transparent);
        }
        mDialog.show();
        if (window != null) {
            window.setLayout(mWidth, mHeight);
        }
        return mDialog;
    }

    private void setLineShowOrHide() {
        View lineHorizontal = mContentView.findViewById(R.id.line_horizontal);
        View lineVertical = mContentView.findViewById(R.id.line_vertival);

        if (!mIfNegativeButtonShown && !mIfPositiveButtonShown) {
            lineHorizontal.setVisibility(View.GONE);
            lineVertical.setVisibility(View.GONE);
        } else if (mIfNegativeButtonShown && mIfPositiveButtonShown) {
            lineHorizontal.setVisibility(View.VISIBLE);
            lineVertical.setVisibility(View.VISIBLE);
        } else {
            lineHorizontal.setVisibility(View.VISIBLE);
            lineVertical.setVisibility(View.GONE);
        }
    }

    public interface OnClickListener {
        /**
         * 下方两个按钮的点击回调
         * @param dialog the showing dialog
         * @param whichButton to show,positive button or negative button
         */
        void onClick(AlertDialog dialog, int whichButton);
    }

}
```





layout_dialog.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
                xmlns:tools="http://schemas.android.com/tools"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:background="@drawable/bg_round_conner"
                android:orientation="vertical">

    <LinearLayout
        android:id="@+id/container_buttons"
        android:layout_width="match_parent"
        android:layout_height="110px"
        android:layout_alignParentBottom="true"
        android:orientation="horizontal"
        tools:ignore="PxUsage">

        <Button
            android:id="@+id/btn_negative"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:gravity="center"
            android:textColor="#FFFFFF"
            android:textSize="36px"
            android:background="@null"
            android:visibility="gone"
            tools:text="取消"/>

        <View
            android:id="@+id/line_vertival"
            android:layout_width="1px"
            android:layout_height="match_parent"
            android:background="#80FFFFFF"/>

        <Button
            android:id="@+id/btn_positive"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1"
            android:gravity="center"
            android:textColor="#FFFFFF"
            android:textSize="36px"
            android:background="@null"
            android:visibility="gone"
            tools:text="确定"/>
    </LinearLayout>

    <View
        android:id="@+id/line_horizontal"
        android:layout_width="match_parent"
        android:layout_height="1px"
        android:layout_above="@id/container_buttons"
        android:background="#80FFFFFF"
        tools:ignore="PxUsage"/>

    <TextView
        android:id="@+id/tv_message"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_above="@id/container_buttons"
        android:gravity="center"
        android:textColor="#FFFFFF"
        android:textSize="36px"
        tools:text="这是一个弹窗"
        tools:ignore="PxUsage"/>

</RelativeLayout>
```





bg_round_conner.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="#48515d" />
    <corners android:topLeftRadius="10px"
             android:topRightRadius="10px"
             android:bottomRightRadius="10px"
             android:bottomLeftRadius="10px"/>
</shape>
```

