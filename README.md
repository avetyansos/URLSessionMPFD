# URLSession Multipart / Form-Data
Code from tutorial on URLSession Multipart / Form-Data

Working with 
Swift version - 5.0 
Network Provider - URLSessionDatatask


Method that creates httpBody for any Request

        func createDataBody(withParameters params: Parameters?, media: [Media]?, boundary: String) -> Data {
        
        let lineBreak = "\r\n"
        var body = Data()
        
        if let parameters = params {
            for (key, value) in parameters {
                body.append("--\(boundary + lineBreak)")
                body.append("Content-Disposition: form-data; name=\"\(key)\"\(lineBreak + lineBreak)")
                body.append("\(value + lineBreak)")
            }
        }
        
        if let media = media {
            for photo in media {
                body.append("--\(boundary + lineBreak)")
                body.append("Content-Disposition: form-data; name=\"\(photo.key)\"; filename=\"\(photo.filename)\"\(lineBreak)")
                body.append("Content-Type: \(photo.mimeType + lineBreak + lineBreak)")
                body.append(photo.data)
                body.append(lineBreak)
            }
        }
        
        body.append("--\(boundary)--\(lineBreak)")
        
        return body
    }

Structure that creates Media type Document

        struct Media {
         let key: String
         let filename: String
         let data: Data
         let mimeType: String
    
         init?(withImage image: UIImage, forKey key: String) {
                 self.key = key
                 self.mimeType = "image/jpeg"
                 self.filename = "kyleleeheadiconimage234567.jpg"
        
                 guard let data = image.jpegData(compressionQuality: 0.7) else { return nil }
                 self.data = data
          }
       }

Create extension thats append Strings to Data 

        extension Data {
            mutating func append(_ string: String) {
                  if let data = string.data(using: .utf8) {
                  append(data)
                }
             }
        }


Method that return baoundry 

    func generateBoundary() -> String {
        return "Boundary-\(NSUUID().uuidString)"
    }


Usage 


        let parameters = ["name": "MyTestFile123321",
                                  "description": "My tutorial test file for MPFD uploads"]
                var mediaImages = [Media]()
                for image in imagesArray {
                    guard let mediaImage = Media(withImage: image, forKey: "image") else { return }
                    mediaImages.append(mediaImage)
                }
        
        guard let url = URL(string: "https://api.imgur.com/3/image") else { return }
        var request = URLRequest(url: url)
        request.httpMethod = "POST"

        let boundary = generateBoundary()

        request.setValue("multipart/form-data; boundary=\(boundary)", forHTTPHeaderField: "Content-Type")
        request.addValue("Client-ID f65203f7020dddc", forHTTPHeaderField: "Authorization")

        let dataBody = createDataBody(withParameters: parameters, media: mediaImages, boundary: boundary)
        request.httpBody = dataBody

        let session = URLSession.shared
        session.dataTask(with: request) { (data, response, error) in
            if let response = response {
                print(response)
            }

            if let data = data {
                do {
                    let json = try JSONSerialization.jsonObject(with: data, options: [])
                    print(json)
                } catch {
                    print(error)
                }
            }
            }.resume()
