---
layout: post
title: "Android ViewHolder pattern for smooth scrolling of ListView"
description: ""
category: tutorial
tags: [android]
---
{% include JB/setup %}


## Sample Listing

    private class MyAdapter extends ArrayAdapter<Object> {

        private final Context mContext;

        public MyAdapter(Context context) {
            super(context, 0);
            mContext = context;
        }

        @Override
        public View getView(int position, View convertView, ViewGroup parent) {
            View view = convertView;
            ViewHolder holder; // to reference the child views for later actions

            // Always check first if convertView already exists or not,
            // so that we can reuse it instead of inflating a new layout every time.
            if (view == null) {
                view = LayoutInflater.from(mContext).inflate(R.layout.my_layout, parent, false);
                
                holder = new ViewHolder();
                holder.view01 = (View) view.findViewById(R.id.view01);
                holder.view02 = (View) view.findViewById(R.id.view02);
                view.setTag(holder);
            } else {
                holder = (ViewHolder) view.getTag();

                // Set new view properties if necessary, or simply return convertView.
                Object obj = getItem(position);

                ...
            }

        return view;
    }

    private static class ViewHolder {
        public View view01;
        public View view02;
    }


## References

* [Making ListView Scrolling Smooth | Android Developers](http://developer.android.com/training/improving-layouts/smooth-scrolling.html)