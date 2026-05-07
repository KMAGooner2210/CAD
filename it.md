            import sys
            import os
            from werkzeug.utils import secure_filename
            from flask import Flask, request, jsonify, send_from_directory
            
            current_dir = os.path.dirname(os.path.abspath(__file__))
            project_root = os.path.dirname(current_dir)
            sys.path.append(project_root)
            
            # Khai báo db
            import backend.database as db
            
            app = Flask(__name__)
            
            # Đảm bảo đường dẫn lưu ảnh trỏ đúng vào thư mục assets của backend
            UPLOAD_FOLDER = os.path.join(project_root, 'backend', 'assets', 'images')
            os.makedirs(UPLOAD_FOLDER, exist_ok=True)
            
            @app.route('/api/stats', methods=['GET'])
            def get_stats(): 
                return jsonify(db.get_stats_data())
            
            @app.route('/api/menu', methods=['GET'])
            def get_menu():
                result = [{"id": p["id"], "name": p["name"], "category": p["category"], "price": p["price"], "image": p["image"] if p["image"] else "#FFD700", "status": bool(p.get("status", 1))} for p in db.get_all_products()]
                return jsonify(result)
            
            @app.route('/images/<filename>')
            def serve_image(filename):
                return send_from_directory(UPLOAD_FOLDER, filename)
            
            @app.route('/api/menu/add', methods=['POST'])
            def add_product():
                name = request.form.get('name')
                category = request.form.get('category')
                price = int(request.form.get('price'))
                
                image_filename = 'default_food.png'
                if 'image_file' in request.files:
                    file = request.files['image_file']
                    if file.filename != '':
                        filename = secure_filename(file.filename)
                        file.save(os.path.join(UPLOAD_FOLDER, filename))
                        image_filename = filename
                        
                # FIX LỖI Ở ĐÂY: Dùng db. thay vì database.
                success = db.admin_add_product(name, category, price, image_filename)
                return jsonify({'success': success})
                
            @app.route('/api/menu/update', methods=['POST'])
            def update_product():
                pid = int(request.form.get('id'))
                name = request.form.get('name')
                category = request.form.get('category')
                price = int(request.form.get('price'))
                
                image_filename = ''
                # FIX LỖI Ở ĐÂY: Sửa image_filename thành image_file
                if 'image_file' in request.files:
                    file = request.files['image_file']
                    if file.filename != '':
                        filename = secure_filename(file.filename)
                        file.save(os.path.join(UPLOAD_FOLDER, filename))
                        image_filename = filename
                
                # FIX LỖI Ở ĐÂY: Dùng db. thay vì database.
                success = db.admin_update_product(pid, name, category, price, image_filename)
                return jsonify({'success': success})
            
            @app.route('/api/menu/delete', methods=['POST'])
            def delete_product():
                return jsonify({"success": db.admin_delete_product(request.json['id'])})
            
            @app.route('/api/menu/toggle', methods=['POST'])
            def toggle_product():
                data = request.json
                return jsonify({"success": db.admin_toggle_product_status(data['id'], data['status'])})
            
            @app.route('/api/report', methods=['GET'])
            def get_report(): 
                return jsonify(db.admin_get_report_data())
            
            @app.route('/api/orders', methods=['GET'])
            def get_orders():
                limit = int(request.args.get('limit', 10))
                page = int(request.args.get('page', 1))
                offset = (page - 1) * limit
                total_items, rows = db.admin_get_orders(request.args.get('status_filter', 'all'), request.args.get('keyword', ''), limit, offset)
                
                import json
                orders = []
                for r in rows:
                    created_at = r['created_at']
                    orders.append({
                        "id": r['order_id'],
                        "date": created_at.split(" ")[0] if created_at else "",
                        "time": created_at.split(" ")[1] if created_at else "",
                        "total": f"{r['total_amount']:,}".replace(",", "."),
                        "status": "paid" if r['payment_status'] == 'PAID' else "cancelled",
                        "items": json.loads(r['items_json']) if r.get('items_json') else []
                    })
                return jsonify({"data": orders, "totalPages": max(1, (total_items + limit - 1) // limit), "currentPage": page, "totalItems": total_items})
            
            if __name__ == '__main__':
                app.run(host='0.0.0.0', port=5000)
