---
layout: post
title: "Creating Android ListView with section headers"
description: ""
category: tutorial
tags: [android, listview]
---
{% include JB/setup %}


## Sample Listing

**SeparatedListAdapter.java**

    public class SeparatedListAdapter extends BaseAdapter {

        private final HeaderAdapter mHeaders;
        private final Map<String, Adapter> mSections = new LinkedHashMap<String, Adapter>();
        private final static int TYPE_SECTION_HEADER = 0;

        public SeparatedListAdapter(Context context) {
            mHeaders = new HeaderAdapter(context);
        }

        public void addSection(String sectionTitle, int count, Adapter adapter) {
            mHeaders.add(new SectionHeader(sectionTitle, count));
            mSections.put(sectionTitle, adapter);
        }

         @Override
        public int getCount() {
            // Add together all sections, plus one for each section header.
            int total = 0;
            for (Adapter adapter : mSections.values())
                total += adapter.getCount() + 1;
            return total;
        }

        @Override
        public int getViewTypeCount() {
            // Assume that headers count as one, then add up all sections.
            int total = 1;
            for (Adapter adapter : mSections.values())
                total += adapter.getViewTypeCount();
            return total;
        }

        @Override
        public Object getItem(int position) {
            for (Object sectionTitle : mSections.keySet()) {
                Adapter adapter = mSections.get(sectionTitle);
                int size = adapter.getCount() + 1;

                // Check if position inside this section.
                if (position == 0)
                    return sectionTitle;
                else if (position < size)
                    return adapter.getItem(position - 1);

                // Otherwise, jump into next section.
                position -= size;
            }
            return null;
        }

        @Override
        public int getItemViewType(int position) {
            int type = 1;
            for (Object sectionTitle : mSections.keySet()) {
                Adapter adapter = mSections.get(sectionTitle);
                int size = adapter.getCount() + 1;

                // Check if position inside this section.
                if (position == 0)
                    return TYPE_SECTION_HEADER;
                else if (position < size)
                    return type + adapter.getItemViewType(position - 1);

                // Otherwise, jump into next section.
                position -= size;
                type += adapter.getViewTypeCount();
            }
            return -1;
        }

        @Override
        public View getView(int position, View convertView, ViewGroup parent) {
            int sectionnum = 0;
            for (Object section : mSections.keySet()) {
                Adapter adapter = mSections.get(section);
                int size = adapter.getCount() + 1;

                // Check if position inside this section.
                if (position == 0)
                    return mHeaders.getView(sectionnum, convertView, parent);
                else if (position < size)
                    return adapter.getView(position - 1, convertView, parent);

                // Otherwise, jump into next section.
                position -= size;
                sectionnum++;
            }
            return null;
        }

        @Override
        public long getItemId(int position) {
            return position;
        }

        public boolean areAllItemsSelectable() {
            return false;
        }

        @Override
        public boolean isEnabled(int position) {
            return (getItemViewType(position) != TYPE_SECTION_HEADER);
        }

        private class SectionHeader {
            private String mTitle;
            private int mCount;

            public SectionHeader(String title, int count) {
                mTitle = title;
                mCount = count;
            }

            public String getTitle() {
                return mTitle;
            }

            public int getCount() {
                return mCount;
            }
        }

        private class HeaderAdapter extends ArrayAdapter<SectionHeader> {
            private final Context mContext;

            public HeaderAdapter(Context context) {
                super(context, 0);
                mContext = context;
            }

            @Override
            public View getView(int position, View convertView, ViewGroup parent) {
                View view = convertView;

                if (view == null) {
                    view = LayoutInflater.from(mContext).inflate(R.layout.list_header, parent, false);
                    HeaderViewHolder holder = new HeaderViewHolder();
                    holder.title = (TextView) view.findViewById(R.id.title);
                    holder.count = (TextView) view.findViewById(R.id.count);
                    view.setTag(holder);
                }

                HeaderViewHolder viewHolder = (HeaderViewHolder) view.getTag();
                SectionHeader header = getItem(position);
                if (viewHolder != null && header != null) {
                    viewHolder.title.setText(header.getTitle());
                    viewHolder.count.setText(String.valueOd(header.getCount());
                }

                return view;
            }
        }

        private static class HeaderViewHolder {
            public TextView title;
            public TextView count;
        }
    }


**list_header.xml**

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:paddingTop="15dp"
        android:orientation="horizontal">

        <TextView
            android:id="@+id/title"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textAppearance="?android:attr/textAppearanceMedium" />

        <TextView
            android:id="@+id/count"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="right"
            android:textAppearance="?android:attr/textAppearanceMedium" />

    </LinearLayout>


## Worth a look

* [HeaderListView for Android](http://applidium.com/en/news/headerlistview_for_android)
* [android-amazing-listview](https://code.google.com/p/android-amazing-listview)


## References

* [Separating Lists with Headers in Android 0.9](http://jsharkey.org/blog/2008/08/18/separating-lists-with-headers-in-android-09)